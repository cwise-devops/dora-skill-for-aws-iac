# Encryption controls

## ENC-01: Encrypt data at rest

**Article**: DORA Art. 9(2); RTS Art. 6(2)(a). Simplified: RTS Art. 35(a). **Basis**: Explicit.

- S3: default encryption with `aws:kms` and an S3 Bucket Key.
- EBS: encryption by default in each account and Region. Encrypt each volume.
- RDS and Aurora: storage encryption on.
- DynamoDB, EFS, SQS, SNS, Kinesis, ElastiCache, OpenSearch and CloudWatch Logs: encryption at rest with a KMS key.
- For Confidential data or a CIF, use a customer managed KMS key (ENC-04). The basis for the key type is Practice.

## ENC-02: Encrypt data in transit, also inside the VPC

**Article**: RTS Art. 6(2)(a), 6(2)(c), 13(e) and 14(1)(a). Simplified: RTS Art. 35(d). **Basis**: Explicit.

RTS Art. 13(e) includes internal networks. TLS at the edge only is not sufficient.

- Load balancers, CloudFront and API Gateway: HTTPS only, with TLS 1.2 or higher. Redirect HTTP to HTTPS.
- S3: add a bucket policy that denies requests when `aws:SecureTransport` is `false`.
- RDS: force TLS in the parameter group (`rds.force_ssl` for PostgreSQL, `require_secure_transport` for MySQL).
- ElastiCache, MSK and OpenSearch: encryption in transit on.
- Traffic between services in the VPC: use TLS to the targets, for example HTTPS target groups.

## ENC-03: Keep cryptographic choices easy to change

**Article**: RTS Art. 6(3) and 6(4). **Basis**: Explicit objective. Practice measure.

RTS Art. 6(4) tells the entity to change cryptographic technology when cryptanalysis makes it weak.

- Put the TLS security policy names in one variable or constant. One change then updates all endpoints.
- Use AWS managed TLS policies that support TLS 1.3, for example `ELBSecurityPolicy-TLS13-1-2-2021-06`.
- CloudFront: use `TLSv1.2_2021` as the minimum protocol version.

## ENC-04: Manage KMS keys through their full lifecycle

**Article**: DORA Art. 9(4)(d); RTS Art. 7(1) and 7(2). **Basis**: Explicit for the lifecycle. Practice for the KMS settings.

- Use customer managed keys with automatic rotation on.
- In the key policy, key administrators and key users are different principals.
- Give `kms:*` to no wildcard principal.
- Set the deletion window to 30 days.
- Use different keys for production and non-production (OPS-01).
- CloudTrail records each use of a key (LOG-01).

## ENC-05: Keep a certificate register and renew certificates before expiry

**Article**: RTS Art. 7(4) and 7(5). **Basis**: Explicit. RTS Art. 7(4) applies at minimum to CIF assets.

- Use ACM certificates with DNS validation. ACM then renews them automatically.
- Add the DORA tags (AST-01) to each certificate. ACM with these tags is the certificate register.
- Add an EventBridge rule for the `ACM Certificate Approaching Expiration` event. Send the event to the alert topic (DET-02).
- For imported certificates, add an alarm on the `DaysToExpiry` metric.

## ENC-06: Protect data in use if the classification makes it necessary

**Article**: RTS Art. 6(2)(b). Simplified: RTS Art. 35(a). **Basis**: Explicit objective. Practice measure.

- Use AWS Nitro Enclaves to process very sensitive data, for example keys or card data.
- If encryption in use is not possible, process the data in a separate account or an isolated subnet (NET-01). RTS Art. 6(2)(b) accepts this measure.

## Out of IaC scope

- The encryption policy, based on the data classification and the risk assessment (RTS Art. 6(1) and 6(2)).
- The record of each mitigation when the entity cannot use leading practice (RTS Art. 6(5)).
- The method to replace a lost or compromised key (RTS Art. 7(3)).
