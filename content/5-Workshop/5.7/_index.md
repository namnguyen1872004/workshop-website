---
title: "SQS, Lambda, and SES"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

The event pipeline is integrated only after the database and media flows are stable. Web requests finalize business operations first, then emit events; Lambda processes asynchronous emails.

5.7.1. SQS Main Queue and DLQ

| Attribute | Workshop Value |
|---|---|
| Main queue | delivery-order-events (Standard) |
| DLQ | delivery-order-events-dlq |
| Visibility timeout | 180 seconds |
| Retention | 4 days |
| Encryption | SQS-managed SSE or KMS per scope |
| maxReceiveCount | 3 |

5.7.2. Event Contract
```json
{
  "schemaVersion": "1.0",
  "eventId": "uuid",
  "eventType": "order.created",
  "occurredAtUtc": "2026-07-22T09:10:14Z",
  "orderId": 6,
  "orderCode": "NF-20260721170553-347",
  "status": "created",
  "customerId": 1,
  "driverId": null
}
```
Events must contain `schemaVersion`, `eventId`, `eventType`, UTC time, and business keys.
Do not embed credentials, cookies, connection strings, or base64 media within messages.
Payloads should only contain metadata needed by the consumer; media should be referenced via object keys if truly needed.

5.7.3. Publisher in ASP.NET Core
1. Register `IAmazonSQS` using an IAM Role; do not hard-code Queue URLs or access keys.
2. Create an `IOrderEventPublisher` and `OrderEventEnvelope`.
3. Publish after `SaveChanges` or when the transaction has committed.
4. Log `eventId`, `eventType`, `orderId`, and AWS MessageId; do not log the entire sensitive payload.
5. In the workshop, you can publish directly; in production, use a Transactional Outbox to prevent gaps between DB commits and SQS sends.

Critical criteria: Manual `SendMessage` only proves the queue infrastructure works. Integration is only complete when a real order action from the app creates a message matching the contract.

5.7.4. Lambda Consumer
The Lambda role needs ReceiveMessage, DeleteMessage, ChangeMessageVisibility, GetQueueAttributes, and the required SES permissions.
The handler must check `eventId` for idempotent processing because SQS can deliver messages more than once.
Only delete messages after successful processing. Errors must be thrown so SQS/Lambda retries and forwards to the DLQ after reaching the limit.
Upon first rollout, you can leave the trigger Disabled, test the handler with a sample payload, and then Enable it.

5.7.5. Amazon SES
```env
SES_FROM_ADDRESS=<VERIFIED_SENDER>
SES_REPLY_TO=<OPTIONAL_REPLY_TO>
APP_BASE_URL=<ALB_OR_CLOUDFRONT_URL>
```
Email sending status: If the SES account is still in the sandbox, only use verified identities/recipients and clearly document the limitation in your report. Do not claim it is a production email setup if production access hasn't been granted.

5.7.6. Pipeline Testing

| Test Layer | Evidence |
|---|---|
| Infrastructure | Manual message sent to the main queue; redrive policy points correctly to the DLQ. |
| Publisher | Create/update a real order and verify the event matches the contract. |
| Lambda | CloudWatch logs show the eventId; no duplicate processing occurs. |
| SES | Send call succeeds or sandbox/identity constraints are clearly documented. |
| Failure | Failed messages retry and move to the DLQ after maxReceiveCount. |