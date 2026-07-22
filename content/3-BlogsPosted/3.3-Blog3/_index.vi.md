---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---


# Chiến lược gộp kết nối trong Amazon Aurora DSQL

Trong quá trình tìm hiểu về cơ sở dữ liệu phân tán trên AWS, tôi đọc được một bài viết về các chiến lược quản lý Connection Pool trong **Amazon Aurora DSQL**.

Khi xây dựng ứng dụng kết nối đến cơ sở dữ liệu, mỗi yêu cầu không nên liên tục tạo một kết nối mới rồi đóng kết nối ngay sau khi truy vấn hoàn tất. Quá trình tạo kết nối mới thường bao gồm TLS Handshake, xác thực IAM và thiết lập phiên làm việc với cơ sở dữ liệu. Nếu thao tác này được lặp lại với tần suất cao, độ trễ của ứng dụng sẽ tăng lên và hệ thống có thể gặp giới hạn về tốc độ tạo kết nối.

Connection Pooling giải quyết vấn đề này bằng cách duy trì một nhóm kết nối đã được thiết lập sẵn. Khi ứng dụng cần thực hiện truy vấn, nó mượn một kết nối từ Pool, sử dụng kết nối đó và trả lại Pool sau khi hoàn tất.

![Blog 3](/workshop-website/images/blog3.png)

## Connection Pooling trong Aurora DSQL

Amazon Aurora DSQL có kiến trúc quản lý kết nối khác với PostgreSQL truyền thống.

Trong PostgreSQL thông thường, một kết nối thường được gắn với một tiến trình Backend trong suốt phiên làm việc. Aurora DSQL sử dụng mô hình Transaction-level Pooling, trong đó kết nối chỉ được ánh xạ đến Query Processor khi có Transaction đang hoạt động.

Điều này giúp một số lượng Query Processor nhỏ hơn có thể phục vụ một số lượng kết nối lớn hơn.

Do khả năng Transaction Multiplexing đã được tích hợp trong kiến trúc dịch vụ, AWS không khuyến nghị sử dụng thêm các Database Proxy phía máy chủ như:

- PgBouncer.
- pgpool-II.
- Các giải pháp Database-side Pooling tương tự.

Việc sử dụng thêm một lớp Proxy có thể làm kiến trúc phức tạp hơn mà không mang lại lợi ích tương ứng.

Aurora DSQL cũng có một số đặc điểm cần chú ý:

- Mỗi kết nối có thời gian tồn tại tối đa một giờ.
- Dịch vụ giới hạn tốc độ tạo kết nối mới.
- Xác thực kết nối sử dụng AWS IAM.
- SQL `PREPARE` và `DEALLOCATE` ở cấp phiên không được hỗ trợ.
- Session-level Advisory Lock và Cursor `WITH HOLD` không được hỗ trợ.

Vì vậy, Connection Pool cần được thiết kế phù hợp với kiến trúc của Aurora DSQL thay vì áp dụng nguyên cấu hình từ PostgreSQL truyền thống.

## Chiến lược 1 – Sử dụng AWS Connector và thư viện Pool phù hợp

AWS khuyến nghị sử dụng các Aurora DSQL Connector chính thức. Connector có thể tự động xử lý vòng đời của IAM Authentication Token, bao gồm tạo Token, lưu trữ tạm thời và làm mới trước khi Token hết hạn.

Một số thư viện Connection Pool phổ biến gồm:

- Java: HikariCP.
- Python: psycopg ConnectionPool.
- Node.js: node-postgres Pool.
- Go: pgxpool.

### Ví dụ với Java và HikariCP

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

Trong cấu hình này:

- `maximumPoolSize` giới hạn số lượng kết nối tối đa.
- `maxLifetime` đặt thời gian tồn tại của kết nối là 55 phút.
- `idleTimeout` đóng những kết nối không cần thiết sau một khoảng thời gian.
- `connectionTimeout` giới hạn thời gian ứng dụng chờ lấy kết nối.
- `keepaliveTime` giúp kiểm tra kết nối còn hoạt động.

### Ví dụ với Python

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

Ứng dụng có thể lấy một kết nối từ Pool như sau:

```python
with pool.connection() as connection:
    result = connection.execute(
        "SELECT * FROM orders WHERE id = %s",
        [order_id]
    )

    order = result.fetchone()
```

Kết nối không bị đóng hoàn toàn sau truy vấn mà được trả lại Pool để tái sử dụng.

## Connection Pooling với AWS Lambda

AWS Lambda có mô hình thực thi khác với ứng dụng chạy liên tục trên Amazon EC2 hoặc Amazon ECS.

Khi một Lambda Execution Environment được tái sử dụng cho nhiều lần gọi, các biến và tài nguyên được tạo ngoài Handler có thể tiếp tục tồn tại. Vì vậy, Connection Pool nên được khởi tạo ở phạm vi Module, không nên tạo mới bên trong Handler.

### Không nên cấu hình như sau

```python
def handler(event, context):
    pool = ConnectionPool(
        conninfo="host=<endpoint> dbname=<database>"
    )

    with pool.connection() as connection:
        return connection.execute("SELECT 1").fetchone()
```

Mỗi lần Lambda chạy có thể tạo một Pool mới, làm tăng số lượng TLS Handshake, IAM Authentication và kết nối đến Aurora DSQL.

### Cách cấu hình phù hợp hơn

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

Pool được tạo một lần khi Execution Environment khởi động và được tái sử dụng trong những lần gọi tiếp theo.

Một số khuyến nghị cho Lambda:

- Khởi tạo Pool bên ngoài Handler.
- Chỉ sử dụng khoảng 1–3 kết nối cho mỗi Execution Environment.
- Không đặt Pool quá lớn vì mỗi Lambda Environment có Pool riêng.
- Đặt thời gian tồn tại kết nối thấp hơn giới hạn một giờ.
- Theo dõi tổng số kết nối khi Lambda có mức Concurrency cao.
- Chấp nhận rằng Cold Start đầu tiên vẫn cần TLS Handshake và IAM Authentication.

Ví dụ, nếu mỗi Lambda Environment có tối đa 2 kết nối và hệ thống có 500 Environment hoạt động đồng thời, tổng số kết nối có thể đạt 1.000.

## Xác thực IAM và bảo mật kết nối

Aurora DSQL sử dụng AWS IAM để xác thực kết nối cơ sở dữ liệu.

Mỗi kết nối mới cần sử dụng một IAM Authentication Token hợp lệ. Nếu sử dụng AWS Connector chính thức, Token có thể được tạo và làm mới tự động.

Nếu ứng dụng tự quản lý Connection Pool, nên tạo Token mới trong Connection Factory hoặc `beforeConnect` Hook cho từng kết nối mới. Không nên tạo một Token rồi lưu trữ quá lâu để dùng cho tất cả kết nối.

Token cũng không thể tồn tại lâu hơn IAM Credential đã sử dụng để tạo nó. Nếu ứng dụng sử dụng một IAM Role Session có thời hạn một giờ thì Token cũng không thể hợp lệ vượt quá thời gian đó.

Kết nối nên được cấu hình:

```text
sslmode=verify-full
```

Chế độ này kiểm tra cả chứng chỉ TLS và danh tính của máy chủ. Các chế độ yếu hơn như `require` chỉ mã hóa kết nối nhưng không xác minh đầy đủ máy chủ đích.

Ngoài ra, IAM Authentication chỉ xác định ai được phép kết nối. Quyền thực hiện câu lệnh SQL vẫn phải được kiểm soát bằng Database Role.

Ứng dụng nên áp dụng nguyên tắc đặc quyền tối thiểu:

- Dịch vụ chỉ đọc chỉ được cấp quyền `SELECT`.
- Dịch vụ ghi dữ liệu chỉ được phép thao tác trên Schema hoặc Table cần thiết.
- Không sử dụng Database Administrator Role trong Connection Pool của ứng dụng.

## Chiến lược 2 – Cấu hình tuổi thọ kết nối và Jitter

Aurora DSQL đóng kết nối khi kết nối đạt tuổi thọ tối đa một giờ, kể cả khi kết nối đang được sử dụng.

Do đó, Connection Pool nên chủ động thay thế kết nối trước giới hạn này. Giá trị khởi đầu phù hợp có thể là khoảng 55 phút.

```text
Maximum connection lifetime: 55 minutes
```

Tuy nhiên, nếu tất cả kết nối trong Pool được tạo gần như cùng lúc, chúng cũng sẽ hết hạn gần như cùng lúc.

Khi hàng trăm hoặc hàng nghìn kết nối đồng loạt bị đóng, ứng dụng sẽ cố gắng tạo lại toàn bộ kết nối. Hiện tượng này được gọi là **Thundering Herd**.

Nó có thể khiến hệ thống vượt giới hạn tốc độ tạo kết nối mới.

Để hạn chế vấn đề này, cần bổ sung Jitter — một khoảng thời gian ngẫu nhiên vào tuổi thọ của từng kết nối.

Ví dụ với Go:

```go
config.MaxConnLifetime = 55 * time.Minute
config.MaxConnLifetimeJitter = 5 * time.Minute
```

Mỗi kết nối sẽ có thời gian tồn tại khác nhau trong một khoảng nhất định, giúp việc tái tạo kết nối được phân bổ đều theo thời gian.

Ngoài ra, các Application Instance không nên được khởi động đồng thời hoàn toàn. Có thể:

- Khởi động theo từng đợt.
- Thêm khoảng chờ ngẫu nhiên.
- Pre-warm Connection Pool trước thời điểm tải cao.
- Sử dụng Rolling Deployment phù hợp.

## Chiến lược 3 – Xác định kích thước Connection Pool

Pool quá nhỏ có thể khiến Request phải chờ kết nối. Pool quá lớn có thể tạo ra nhiều kết nối không cần thiết và gây khó khăn khi hệ thống mở rộng theo chiều ngang.

Một cấu hình khởi đầu có thể gồm:

| Tham số | Giá trị tham khảo |
|---|---:|
| Minimum Pool Size | 2–5 kết nối |
| Maximum Pool Size | 10–20 kết nối cho mỗi Application Instance |
| Connection Timeout | Khoảng 30 giây |
| Maximum Lifetime | Khoảng 55 phút |
| Idle Timeout | Tắt hoặc phù hợp với Maximum Lifetime |

Các giá trị trên chỉ là điểm bắt đầu. Cấu hình thực tế cần dựa trên:

- Số lượng Request đồng thời.
- Thời gian thực hiện Transaction.
- Số Application Instance.
- Mức độ sử dụng Connection.
- Độ trễ chấp nhận được.
- Số lượng kết nối tối đa của Cluster.

Thay vì tăng Pool của một Application Instance lên quá lớn, nên cân nhắc mở rộng theo chiều ngang:

```text
Nhiều Application Instance + Pool nhỏ
```

thay vì:

```text
Một Application Instance + Pool rất lớn
```

Cách này thường giúp phân phối tải tốt hơn và giảm ảnh hưởng khi một Instance gặp sự cố.

## Chiến lược 4 – Connection Pooling trong Multi-Region

Aurora DSQL hỗ trợ kiến trúc Multi-Region Active-Active.

Trong mô hình này:

- Câu lệnh SQL được xử lý tại Region gần ứng dụng.
- Read Transaction được thực hiện cục bộ.
- Write Transaction chỉ phát sinh độ trễ liên vùng khi thực hiện `COMMIT`.
- Không tồn tại khái niệm Primary Region cố định giống một số hệ thống cơ sở dữ liệu truyền thống.

Ứng dụng nên tạo Connection Pool riêng cho từng Region:

```text
Ứng dụng tại Region A
        ↓
Pool A
        ↓
Aurora DSQL Endpoint A
```

```text
Ứng dụng tại Region B
        ↓
Pool B
        ↓
Aurora DSQL Endpoint B
```

Không nên để ứng dụng ở Region A thường xuyên kết nối đến Endpoint của Region B nếu không cần thiết.

Ngoài ra, cần bảo đảm:

- IAM Token được tạo đúng Region.
- Ứng dụng sử dụng Endpoint cục bộ.
- Mỗi Region có cấu hình Pool phù hợp với lưu lượng riêng.
- Cơ chế Failover và Retry không tạo ra quá nhiều kết nối mới cùng lúc.

## Theo dõi Connection Pool

Chỉ theo dõi Database Metric là chưa đủ.

Amazon CloudWatch có thể cho biết số lượng Transaction hoặc Commit Latency, nhưng không cho biết ứng dụng có đang chờ lấy kết nối từ Pool hay không.

Do đó, cần xuất thêm các Metric phía ứng dụng.

### Pool Acquire Latency

Đây là thời gian Request phải chờ để lấy được kết nối.

Nếu P95 Acquire Latency tăng cao, Pool có thể đã quá nhỏ hoặc Transaction đang giữ kết nối quá lâu.

### Active Connection Count

Cho biết số kết nối đang được sử dụng.

Nếu giá trị này liên tục bằng Maximum Pool Size, hệ thống có nguy cơ Pool Exhaustion.

### Idle Connection Count

Cho biết số kết nối sẵn sàng phục vụ Request mới.

Nếu số lượng kết nối nhàn rỗi luôn bằng 0, Pool có thể không đủ khả năng xử lý tải tăng đột biến.

### Waiting Request Count

Cho biết số Request đang chờ lấy kết nối.

Giá trị này tăng liên tục là dấu hiệu cần:

- Tăng Pool Size hợp lý.
- Giảm thời gian Transaction.
- Tăng số Application Instance.
- Tối ưu câu truy vấn.

### Connection Creation Rate

Theo dõi số kết nối mới được tạo mỗi giây.

Nếu giá trị tiến gần giới hạn của Aurora DSQL, cần kiểm tra:

- Connection Lifetime.
- Jitter.
- Tần suất Restart Application.
- Idle Timeout.
- Pool Pre-warming.

Các thư viện thường cung cấp sẵn Pool Statistic:

- HikariCP: `HikariPoolMXBean`.
- psycopg: `pool.get_stats()`.
- node-postgres: `totalCount`, `idleCount`, `waitingCount`.
- pgxpool: `pool.Stat()`.

Các Metric này có thể được gửi đến Amazon CloudWatch, Prometheus, Grafana hoặc công cụ Application Performance Monitoring.

## Xử lý một số lỗi phổ biến

### Pool Exhaustion

**Biểu hiện:**

- Request bị Timeout khi lấy Connection.
- Waiting Request tăng cao.
- Active Connection luôn đạt mức tối đa.

**Cách xử lý:**

- Tăng Maximum Pool Size có kiểm soát.
- Tăng số Application Instance.
- Rút ngắn Transaction.
- Tối ưu Query.
- Tránh giữ Connection trong khi thực hiện tác vụ không liên quan đến Database.

### Lỗi xác thực sau một khoảng thời gian

**Nguyên nhân có thể:**

- Token hết hạn.
- Connection sống quá lâu.
- Ứng dụng không làm mới IAM Token.

**Cách xử lý:**

- Sử dụng AWS Connector chính thức.
- Tạo Token mới cho từng kết nối mới.
- Đặt Maximum Lifetime khoảng 55 phút.
- Kiểm tra thời hạn của IAM Credential.

### Lỗi tốc độ tạo kết nối

**Nguyên nhân có thể:**

- Tất cả Connection hết hạn cùng lúc.
- Nhiều Application Instance khởi động đồng thời.
- Idle Timeout quá ngắn.
- Lambda Concurrency tăng đột biến.

**Cách xử lý:**

- Thêm Jitter.
- Khởi động Instance theo từng đợt.
- Giữ lại kết nối nhàn rỗi thay vì liên tục đóng và tạo mới.
- Pre-warm Pool trước thời gian tải cao.

## Danh sách cấu hình khuyến nghị

Trước khi đưa ứng dụng vào Production, nên kiểm tra:

- Sử dụng AWS Connector hoặc thư viện Pool phù hợp.
- Bật `sslmode=verify-full`.
- Tạo IAM Token mới cho mỗi kết nối mới.
- Áp dụng Database Role theo nguyên tắc đặc quyền tối thiểu.
- Đặt Maximum Connection Lifetime thấp hơn một giờ.
- Sử dụng Jitter khi thay thế kết nối.
- Không sử dụng PgBouncer hoặc pgpool-II.
- Cấu hình Pool nhỏ cho từng AWS Lambda Environment.
- Tạo Pool ngoài Lambda Handler.
- Theo dõi Acquire Latency, Active, Idle và Waiting Connection.
- Theo dõi Connection Creation Rate.
- Tạo Pool riêng cho từng Region trong mô hình Multi-Region.
- Chuẩn bị Retry và Backoff khi kết nối thất bại.

## Kết luận

Qua bài viết này, tôi hiểu rõ hơn rằng Connection Pooling trong Amazon Aurora DSQL không chỉ đơn giản là tái sử dụng kết nối.

Pool cần được cấu hình phù hợp với:

- Transaction-level Multiplexing của Aurora DSQL.
- AWS IAM Authentication.
- Giới hạn tuổi thọ kết nối một giờ.
- Tốc độ tạo kết nối mới.
- Kiến trúc AWS Lambda.
- Mô hình Multi-Region Active-Active.

Bốn chiến lược quan trọng gồm sử dụng Connector phù hợp, quản lý tuổi thọ kết nối với Jitter, xác định kích thước Pool hợp lý và tạo Pool riêng theo từng Region.

Theo tôi, điểm quan trọng nhất là không nên chỉ theo dõi Database. Connection Pool nằm trong ứng dụng nên cũng cần được đo lường và cảnh báo như một thành phần của hệ thống.

Khi được cấu hình và giám sát đúng, Connection Pool giúp giảm TLS Handshake, hạn chế chi phí xác thực IAM, tránh Connection Storm và duy trì độ trễ ổn định khi ứng dụng mở rộng.

<p align="center">
  <strong>Bài viết tham khảo:</strong>
  <a href="https://aws.amazon.com/vi/blogs/database/connection-pooling-strategies-in-amazon-aurora-dsql/" target="_blank">
    AWS Database Blog
  </a>
</p>