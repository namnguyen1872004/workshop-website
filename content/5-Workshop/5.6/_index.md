---
title: "S3 for POD, Signatures, and Evidence"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

S3 is integrated before SQS to finalize the business flow and eliminate local disk dependencies. When an EC2 instance is replaced, media remains intact and the application can scale.

5.6.1. Bucket, Encryption, and Prefixes
Create a bucket `delivery-dev-pod-<unique-suffix>` in `ap-southeast-1`, enable Block Public Access, and default encryption. For the workshop, you can use SSE-S3; if using SSE-KMS, you must grant the correct key permissions to the EC2 role.

| Data Type | Standard Object Key |
|---|---|
| POD Image | pod/{orderId}/image/{guid}.png |
| Signature | pod/{orderId}/signature/{guid}.png |
| Failed Delivery Evidence | failed-evidence/{orderId}/{guid}.png |
| Deployment Package | deployments/{release}.tar.gz |

5.6.2. S3 Gateway VPC Endpoint
Create a Gateway Endpoint `com.amazonaws.ap-southeast-1.s3` and associate it with `delivery-app-rt`. Private EC2 instances access S3 without routing S3 traffic through NAT.
Route check: The endpoint must be attached to the correct application route table; bucket policies/KMS policies still must allow the respective IAM Role.

5.6.3. Application Changes
```env
Storage__BucketName=delivery-dev-pod-<unique-suffix>
AWS_REGION=ap-southeast-1
```
Register `IAmazonS3` using the default credential provider chain.
Create an `IMediaStorage` service to upload, read, and delete by prefix; controllers should not call the SDK directly.
The database only stores object keys, content types, sizes, and necessary checksums; do not store public URLs or base64 strings.
Endpoints `/media/...` must verify access permissions and then either stream the object or generate a short-lived presigned URL.
If a DB transaction fails after an upload, there must be a compensating delete or a cleanup job for orphaned objects.

5.6.4. Testing
```bash
printf "delivery-s3-test" | sudo -u deliveryapp aws s3 cp - s3://<BUCKET_NAME>/pod/_test/connectivity.txt --region ap-southeast-1
sudo -u deliveryapp aws s3api head-object --bucket <BUCKET_NAME> --key pod/_test/connectivity.txt --region ap-southeast-1 --query '{Size:ContentLength,Encryption:ServerSideEncryption,KmsKey:SSEKMSKeyId}'
sudo -u deliveryapp aws s3 rm s3://<BUCKET_NAME>/pod/_test/connectivity.txt --region ap-southeast-1
```

| Scenario | Passing Evidence |
|---|---|
| Successful Delivery | DB contains PodImagePath and DriverSignaturePath; S3 contains the two corresponding objects. |
| Failed Delivery | Evidence resides under `failed-evidence/{orderId}/`. |
| Security | Bucket is not public; objects are not accessible via public URLs. |
| Scale/replace EC2 | Media remains readable after instance replacement. |