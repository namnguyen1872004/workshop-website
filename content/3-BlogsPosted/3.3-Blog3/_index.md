---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---


# Connection pooling strategies in Amazon Aurora DSQL

While learning about distributed database services on AWS, I found an AWS Database Blog article describing connection pooling strategies for **Amazon Aurora DSQL**.

Applications should not establish and close a new database connection for every request. Creating a connection requires a TLS handshake, IAM authentication, and database-session initialization. Repeating this process at high frequency increases latency and can cause the application to reach service connection-rate limits.

Connection pooling solves this problem by maintaining a reusable collection of established connections. When the application needs to run a query, it borrows a connection, performs the transaction, and returns the connection to the pool.

![Blog 3](/workshop-website/images/blog3.png)

## Connection Pooling in Aurora DSQL

Amazon Aurora DSQL manages connections differently from a traditional PostgreSQL deployment.

In standard PostgreSQL, a client connection commonly maintains a persistent relationship with a backend server process. Aurora DSQL uses transaction-level pooling. A connection is assigned to a Query Processor only while an active transaction is being executed.

This architecture allows a smaller number of Query Processors to support a larger number of mostly idle client connections.

Because transaction multiplexing is built into the service, AWS does not recommend adding database-side pooling proxies such as:

- PgBouncer.
- pgpool-II.
- Similar server-side connection-pooling solutions.

An additional proxy layer can duplicate capabilities already provided by the service and increase architectural complexity.

Important Aurora DSQL connection characteristics include:

- Connections have a maximum age of one hour.
- The service enforces a new-connection rate limit.
- Database authentication is based on AWS IAM.
- SQL-level `PREPARE` and `DEALLOCATE` statements are not supported.
- Session-level advisory locks and `WITH HOLD` cursors are not supported.

The connection pool must therefore be designed specifically for Aurora DSQL instead of reusing a traditional PostgreSQL configuration without adjustment.

## Strategy 1 – Use AWS Connectors and an Appropriate Pool Library

AWS recommends using official Aurora DSQL connectors. They manage the IAM authentication-token lifecycle, including token generation, caching, and transparent refresh.

Common client-side pool libraries include:

- Java: HikariCP.
- Python: psycopg ConnectionPool.
- Node.js: node-postgres Pool.
- Go: pgxpool.

### Java with HikariCP

```java
HikariConfig config = new HikariConfig();

config.setJdbcUrl(
    "jdbc:postgresql://<cluster-endpoint>:5432/<database>"
    + "?ssl=true"
    + "&sslmode=verify-full"
    + "&sslrootcert=<certificate-path>"
);

config.setMaximumPoolSize(20);
config.setMaxLifetime(55 * 60 * 1000);
config.setIdleTimeout(10 * 60 * 1000);
config.setConnectionTimeout(30 * 1000);
config.setKeepaliveTime(5 * 60 * 1000);

HikariDataSource dataSource = new HikariDataSource(config);
```

The main parameters include:

- `maximumPoolSize`: maximum number of connections.
- `maxLifetime`: maximum age of each connection.
- `idleTimeout`: how long an unused connection remains in the pool.
- `connectionTimeout`: how long a request waits for a connection.
- `keepaliveTime`: how often an idle connection is checked.

### Python with psycopg

```python
from psycopg_pool import ConnectionPool

pool = ConnectionPool(
    conninfo=(
        "host=<cluster-endpoint> "
        "dbname=<database> "
        "sslmode=verify-full"
    ),
    min_size=2,
    max_size=20,
    max_lifetime=55 * 60,
    max_idle=10 * 60,
    reconnect_timeout=30,
)
```

The application can borrow a connection as follows:

```python
with pool.connection() as connection:
    result = connection.execute(
        "SELECT * FROM orders WHERE id = %s",
        [order_id]
    )

    order = result.fetchone()
```

After the operation completes, the connection is returned to the pool instead of being permanently closed.

## Connection Pooling with AWS Lambda

AWS Lambda uses an execution model that differs from continuously running applications on Amazon EC2 or Amazon ECS.

When Lambda reuses an execution environment, objects created outside the handler can remain available for subsequent invocations. The connection pool should therefore be created at module scope instead of inside the handler.

### Avoid this pattern

```python
def handler(event, context):
    pool = ConnectionPool(
        conninfo="host=<endpoint> dbname=<database>"
    )

    with pool.connection() as connection:
        return connection.execute("SELECT 1").fetchone()
```

This pattern can create a new pool repeatedly, increasing TLS handshakes, IAM authentication operations, and connections to Aurora DSQL.

### Recommended pattern

```python
from psycopg_pool import ConnectionPool

pool = ConnectionPool(
    conninfo=(
        "host=<cluster-endpoint> "
        "dbname=<database> "
        "sslmode=verify-full"
    ),
    min_size=1,
    max_size=2,
    max_lifetime=55 * 60,
    reconnect_timeout=30,
)

def handler(event, context):
    with pool.connection() as connection:
        result = connection.execute(
            "SELECT * FROM orders WHERE id = %s",
            [event["order_id"]]
        )

        return result.fetchone()
```

The pool is initialized once for the execution environment and reused across warm invocations.

Important Lambda recommendations include:

- Instantiate the pool outside the handler.
- Use approximately 1–3 connections per execution environment.
- Avoid large pools because each concurrent Lambda environment has its own pool.
- Recycle connections before the one-hour limit.
- Monitor total connection counts during high Lambda concurrency.
- Account for TLS and IAM authentication during cold starts.

For example, 500 concurrent Lambda execution environments with two connections each can create up to 1,000 database connections.

## IAM Authentication and Connection Security

Amazon Aurora DSQL uses AWS IAM for database authentication.

Each new connection requires a valid IAM authentication token. Official AWS connectors automate token generation and refresh.

When implementing a custom connection pool, generate a new token through the connection factory or a `beforeConnect` hook whenever the pool creates a new connection. Avoid manually caching one token for long periods.

A token cannot remain valid longer than the IAM credentials used to create it. If an application assumes a role with a one-hour session, the generated database token cannot outlive that session.

Connections should use:

```text
sslmode=verify-full
```

This mode validates both the TLS certificate and the identity of the server. Weaker modes might encrypt the connection without fully verifying the destination server.

IAM authentication determines who can connect, but database roles determine what the connected session can do.

Apply least privilege:

- Read-only services receive only `SELECT` permissions.
- Write services receive access only to required schemas or tables.
- Application pools should not use database-administrator roles.

## Strategy 2 – Configure Connection Lifetime with Jitter

Aurora DSQL enforces a hard maximum connection age of one hour.

A pool should recycle connections before they reach that limit. A useful starting value is approximately 55 minutes.

```text
Maximum connection lifetime: 55 minutes
```

However, connections created at approximately the same time will also expire at approximately the same time.

If hundreds or thousands of connections are closed together, application instances can attempt to recreate them simultaneously. This is known as a **thundering-herd reconnection storm**.

It can cause the application to exceed the new-connection rate limit.

Add random jitter to each connection’s maximum lifetime.

Example with Go:

```go
config.MaxConnLifetime = 55 * time.Minute
config.MaxConnLifetimeJitter = 5 * time.Minute
```

Connections are then recycled at different times instead of all at once.

Additional techniques include:

- Starting application instances in stages.
- Adding randomized startup delay.
- Pre-warming pools before expected traffic peaks.
- Using controlled rolling deployments.

## Strategy 3 – Size the Connection Pool Correctly

A pool that is too small causes requests to wait. A pool that is too large creates unnecessary connections and can make horizontal scaling more difficult.

A starting configuration can include:

| Parameter | Suggested starting value |
|---|---:|
| Minimum pool size | 2–5 connections |
| Maximum pool size | 10–20 per application instance |
| Connection timeout | Approximately 30 seconds |
| Maximum lifetime | Approximately 55 minutes |
| Idle timeout | Disabled or aligned with maximum lifetime |

These are starting values rather than universal requirements.

Actual sizing depends on:

- Request concurrency.
- Transaction duration.
- Number of application instances.
- Connection utilization.
- Acceptable latency.
- Cluster connection quotas.

Prefer:

```text
Multiple application instances + smaller pools
```

over:

```text
One application instance + one very large pool
```

Horizontal scaling generally distributes load more effectively and reduces the impact of an individual instance failure.

## Strategy 4 – Multi-Region Pooling

Amazon Aurora DSQL supports multi-Region active-active deployments.

In this architecture:

- SQL processing occurs in the Region close to the client.
- Read transactions remain local.
- Read-write transactions incur cross-Region latency mainly during `COMMIT`.
- The service does not expose a fixed primary Region.

Create a separate connection pool for each Region:

```text
Application in Region A
        ↓
Regional Pool A
        ↓
Aurora DSQL Endpoint A
```

```text
Application in Region B
        ↓
Regional Pool B
        ↓
Aurora DSQL Endpoint B
```

Applications should normally connect to the local endpoint rather than sending routine database traffic to another Region.

Also verify that:

- IAM tokens are created for the correct Region.
- Each application uses its local Aurora DSQL endpoint.
- Pool sizing reflects regional traffic.
- Failover and retry logic do not create a connection storm.

## Monitor the Connection Pool

Database metrics alone are not enough.

Amazon CloudWatch can show transaction activity and commit latency, but it cannot directly reveal how long an application waits for a connection from its pool.

Instrument client-side pool metrics.

### Pool Acquire Latency

Measures how long a request waits to obtain a connection.

High P95 acquire latency can indicate pool saturation or long-running transactions.

### Active Connection Count

Shows how many connections are executing transactions.

A value that remains at the maximum pool size indicates a risk of exhaustion.

### Idle Connection Count

Shows how many connections are immediately available.

A pool with no idle connections has limited capacity for traffic bursts.

### Waiting Request Count

Shows the number of requests waiting for an available connection.

A sustained increase can require:

- A controlled pool-size increase.
- Shorter transactions.
- More application instances.
- Query optimization.

### Connection Creation Rate

Measures how many connections the application creates per second.

A high rate can indicate:

- Connections expiring together.
- Frequent application restarts.
- Idle timeout that is too short.
- Sudden Lambda concurrency growth.

Pool libraries normally expose these statistics:

- HikariCP: `HikariPoolMXBean`.
- psycopg: `pool.get_stats()`.
- node-postgres: `totalCount`, `idleCount`, and `waitingCount`.
- pgxpool: `pool.Stat()`.

Publish these metrics to Amazon CloudWatch, Prometheus, Grafana, or an application-performance monitoring platform.

## Troubleshooting Common Issues

### Pool Exhaustion

**Symptoms:**

- Timeouts while acquiring a connection.
- Growing waiting-request count.
- Active connections remain at the configured maximum.

**Actions:**

- Increase the maximum pool size carefully.
- Add application instances.
- Shorten transactions.
- Optimize SQL queries.
- Do not hold a database connection while performing unrelated work.

### Authentication Failures After a Period of Time

**Possible causes:**

- Expired IAM token.
- Connections exceeding their lifetime.
- Missing token-refresh logic.

**Actions:**

- Use an official AWS connector.
- Generate a fresh token for every new connection.
- Set maximum lifetime to approximately 55 minutes.
- Verify the lifetime of the underlying IAM credentials.

### New-Connection Rate Errors

**Possible causes:**

- Connections expire at the same time.
- Application instances start simultaneously.
- Idle timeout is too aggressive.
- Lambda concurrency increases rapidly.

**Actions:**

- Add connection-lifetime jitter.
- Stagger application starts.
- Keep useful idle connections open.
- Pre-warm the pool before expected traffic increases.

## Production Checklist

Before deploying to production, verify that the application:

- Uses a supported AWS connector or pool library.
- Enables `sslmode=verify-full`.
- Generates a new IAM token for every new connection.
- Applies least-privilege database roles.
- Sets connection lifetime below one hour.
- Uses jitter during connection recycling.
- Does not use PgBouncer or pgpool-II.
- Uses small pools in AWS Lambda execution environments.
- Creates the Lambda pool outside the handler.
- Monitors acquire latency, active, idle, and waiting connections.
- Monitors the new-connection creation rate.
- Creates separate pools for each Region.
- Uses retry and exponential-backoff logic for connection failures.

## Conclusion

This article helped me understand that connection pooling in Amazon Aurora DSQL involves more than simply reusing established database connections.

The pool must account for:

- Aurora DSQL transaction-level multiplexing.
- AWS IAM authentication.
- The one-hour connection-age limit.
- The new-connection rate limit.
- AWS Lambda execution environments.
- Multi-Region active-active architecture.

The four key strategies are selecting an appropriate connector, managing connection lifetime with jitter, sizing pools carefully, and creating regional pools for multi-Region deployments.

In my opinion, one of the most important lessons is that the connection pool must be monitored as an application component. Database metrics alone cannot identify pool acquire latency, waiting requests, or connection churn.

A properly configured and observable pool reduces TLS and authentication overhead, avoids reconnection storms, and helps maintain predictable application latency as the workload scales.

<p align="center">
  <strong>Reference Article:</strong>
  <a href="https://aws.amazon.com/blogs/database/connection-pooling-strategies-in-amazon-aurora-dsql/" target="_blank">
    AWS Database Blog
  </a>
</p>