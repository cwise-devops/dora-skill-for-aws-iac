# Backup and recovery controls

## BCK-01: Back up data at a frequency that agrees with its criticality

**Article**: DORA Art. 12(1)(a) and 16(1)(f). Simplified: RTS Art. 39(2)(g). **Basis**: Explicit.

- Use AWS Backup plans. Set the schedule from the RPO variable.
- Select the resources by the tags `dora:cif` and `dora:classification`.
- Turn on point-in-time recovery for RDS, Aurora and DynamoDB if the RPO is short.
- Turn on S3 versioning for buckets with CIF data.

## BCK-02: Keep backups in a separate account and protect them

**Article**: DORA Art. 12(2) and 12(3). **Basis**: Explicit for the segregation. Practice for the AWS measures.

DORA Art. 12(3) tells the entity to restore on systems that are physically and logically segregated from the source system.

- Copy each backup to a vault in a separate backup account in the organisation.
- Use AWS Backup Vault Lock in compliance mode on that vault.
- Add a vault access policy that denies the deletion of recovery points.
- Encrypt the vault with a KMS key in the backup account.

## BCK-03: Restore in a segregated environment and test restores

**Article**: DORA Art. 11(6), 12(2) and 12(3). Simplified: RTS Art. 40(1). **Basis**: Explicit.

- Add AWS Backup restore testing plans for CIF resources.
- Restore into an isolated account or VPC, not into production.
- Add a validation step, for example a Lambda function that examines the restored data.

## BCK-04: Keep redundant capacity for critical components

**Article**: DORA Art. 11(5) and 12(4). **Basis**: Explicit. Microenterprises must assess the need for redundant capacity.

- Use Multi-AZ for RDS, Aurora, ElastiCache and OpenSearch.
- Spread Auto Scaling groups, ECS services and EKS node groups across at least 2 Availability Zones.
- Use one NAT gateway and one interface endpoint subnet for each Availability Zone.
- For a CIF, use no single-AZ resource.

## BCK-05: Design for the RTO and the RPO

**Article**: DORA Art. 12(6); RTS Art. 24(1)(b)(ii). **Basis**: Explicit for the objectives. Practice for the DR pattern.

- Select the DR pattern from the RTO and RPO variables: backup and restore, pilot light, warm standby, or active-active.
- For a multi-Region design, use Aurora Global Database, DynamoDB global tables, S3 replication and cross-Region backup copies.
- Use Route 53 failover routing or Route 53 Application Recovery Controller.
- Use only data plane actions for failover, for example Route 53 health checks or Application Recovery Controller routing controls.
- The control planes of IAM, Route 53 and CloudFront are in `us-east-1`. A failover that changes these services depends on that Region.
- Make sure that the secondary site does not share a failure domain with the primary site. A second Availability Zone does not protect against a Region event.

Some entity types have stricter limits:

| Entity type | Limit | Article |
|---|---|---|
| Central counterparty | Recovery time for critical functions of 2 hours or less. A secondary processing site with a different geographical risk profile. | RTS Art. 24(2) |
| Central securities depository | RTO for CIF of 2 hours or less. A secondary processing site at a geographical distance. | RTS Art. 24(3); DORA Art. 12(5) |
| Trading venue | Trading starts again within or close to 2 hours. Data loss close to zero. | RTS Art. 24(4) |

For these entity types, use a second Region as the secondary processing site.

## BCK-06: Make the switchover testable

**Article**: DORA Art. 11(6), second subparagraph; RTS Art. 25(2)(c). **Basis**: Explicit. This control does not apply to microenterprises.

- Control the failover with one IaC variable or with Route 53 Application Recovery Controller routing controls.
- Add AWS Fault Injection Service experiment templates for the loss of an instance, an Availability Zone or a dependency.
- Write the failover and failback steps as SSM Automation documents.

## BCK-07: Find single points of failure

**Article**: DORA Art. 10(1). **Basis**: Explicit objective. Practice measure.

- Define each CIF workload as an application in AWS Resilience Hub.
- Give the application a resiliency policy that uses the RTO and RPO variables.

## Out of IaC scope

- The backup policy: the scope and the minimum frequency (DORA Art. 12(1)(a)).
- The business impact analysis (DORA Art. 11(5)).
- The ICT business continuity policy and the response and recovery plans (RTS Art. 24 and 26).
- The yearly test of the plans and the record of the results (DORA Art. 11(6); RTS Art. 25(5)).
- The data integrity checks and reconciliations after a recovery (DORA Art. 12(7)).
