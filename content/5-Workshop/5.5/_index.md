---
title: "RDS MySQL, Secrets Manager, and Migration"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The database is integrated immediately after the runtime is stable because all business logic, S3 object keys, and event metadata rely on accurate data.

5.5.1. DB Subnet Group and RDS
Create `delivery-dev-db-subnet-group` from two database subnets in two Availability Zones. RDS MySQL 8.4 uses private access, Single-AZ for the workshop, and `delivery-rds-sg`.
Scope: Single-AZ is a cost-reduction choice for the workshop; do not present it as a Multi-AZ production setup unless actually enabled.

5.5.2. Secret Contract
```json
{
  "engine": "mysql",
  "host": "<RDS_ENDPOINT>",
  "port": 3306,
  "username": "<DB_USER>",
  "password": "<SECRET_VALUE>",
  "dbname": "nightfury"
}
```
```env
Database__RdsSecretId=<RDS_SECRET_ARN>
Database__Name=nightfury
AWS_REGION=ap-southeast-1
```
The application uses the default AWS credential provider chain. The IAM Role requires `secretsmanager:GetSecretValue` and `DescribeSecret` on the exact secret ARN; if the secret uses a customer-managed KMS key, append `kms:Decrypt` for that key.

5.5.3. Safe Migration Process
1. Create a manual snapshot and logical backup before any risky schema changes.
2. Cross-check models, migration files, and the `__EFMigrationsHistory` table.
3. Only create a new migration when the model actually changes; do not create migrations just to fix endpoint or credential errors.
4. Publish the correct runtime, load `/etc/delivery/delivery.env`, and run the migration as the `deliveryapp` user.
5. Check the migration history, record counts, and main business queries.

```bash
sudo -u deliveryapp bash -c 'set -a; . /etc/delivery/delivery.env; set +a; cd /opt/delivery/current; ./WedNightFury --migrate'
```

5.5.4. Testing and Rollback
```bash
sudo journalctl -u delivery.service --since "10 minutes ago" --no-pager | grep -Ei 'RdsSecretProvider|migrat|fail|error'
mysql -h <RDS_ENDPOINT> -u <DB_USER> -p -e "USE nightfury; SELECT * FROM __EFMigrationsHistory;"
```

| Verification Point | Expectation |
|---|---|
| Secret | The application reads the secret successfully using the IAM Role. |
| Network | RDS Public access = No; 3306 allowed only from EC2 SG. |
| Migration | Schema up-to-date; no duplicate columns/tables. |
| Data | The milestone of 3 users, 5 hubs, and 13 orders remains intact if using the correct DB. |
| Rollback | Snapshots/backups exist, and you know which release/migration to roll back to. |