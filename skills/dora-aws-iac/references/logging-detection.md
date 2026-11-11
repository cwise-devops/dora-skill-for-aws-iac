# Logging and detection controls

## LOG-01: Log API activity and access events

**Article**: RTS Art. 12(2)(c)(i), (iii) and (iv). Simplified: RTS Art. 34(f). **Basis**: Explicit objective. Practice for the AWS measures.

- Use one organisation trail in CloudTrail for all accounts and all Regions.
- Log management events, both read and write.
- Log data events for S3 buckets, DynamoDB tables and Lambda functions that hold or process CIF data.

## LOG-02: Log network traffic

**Article**: RTS Art. 12(2)(c)(v). **Basis**: Explicit objective. Practice for the AWS measures.

- Turn on VPC Flow Logs for all traffic in each VPC.
- Turn on access logs for load balancers, CloudFront and API Gateway.
- Turn on logs for AWS WAF, AWS Network Firewall and Route 53 Resolver queries.

## LOG-03: Log changes, operations and capacity

**Article**: RTS Art. 12(2)(c)(ii), (iii) and (iv). Simplified: RTS Art. 34(f). **Basis**: Explicit objective. Practice for the AWS measures.

- Turn on the AWS Config recorder for all resources in each account and Region. It records each change.
- Send operating system and application logs to CloudWatch Logs.
- Turn on EKS control plane logs and RDS log exports.
- Collect capacity metrics for CIF resources (OPS-07).

## LOG-04: Protect logs against change, deletion and unauthorised access

**Article**: RTS Art. 12(2)(d), 22(d) and 23(3). **Basis**: Explicit for the protection. Practice for the AWS measures.

- Send all logs to a central log archive account.
- Use S3 Object Lock on the log archive buckets.
- Turn on CloudTrail log file validation.
- Encrypt the logs with a KMS key in the log archive account.
- Use SCPs to deny `cloudtrail:StopLogging`, `cloudtrail:DeleteTrail` and `config:StopConfigurationRecorder` in member accounts.

## LOG-05: Set log retention from the risk assessment

**Article**: RTS Art. 12(2)(a) and 22(d). **Basis**: Explicit objective. Practice for the AWS measures.

DORA and the RTS give no retention period. The entity sets it from its business objectives, security objectives and risk assessment.

- Compare the configured retention with the entity-approved log retention policy. A documented, approved constant is valid evidence.
- For unresolved periods or reusable modules, use required input variables with no default value. An input variable alone does not prove that retention is adequate.
- Apply the approved periods to CloudWatch Logs retention, S3 lifecycle rules and Object Lock retention. Record Unknown if the approved policy or effective value is unavailable; record a Gap if the value contradicts the policy.

RTS Art. 22(d) refers to Art. 15 of RTS 2024/1772. The published RTS 2024/1772 has no Art. 15. Refer to RTS Art. 22(d) only.

## LOG-06: Detect a failure of logging

**Article**: RTS Art. 12(2)(e) and 23(2), second subparagraph. **Basis**: Explicit objective. Practice for the AWS measures.

- Send an alert for the CloudTrail events `StopLogging`, `DeleteTrail`, `UpdateTrail` and `PutEventSelectors`.
- Add the AWS Config rules `cloudtrail-enabled` and `multi-region-cloudtrail-enabled`.
- Add an alarm when a critical log group gets no new events. Treat missing data as breaching.

## LOG-07: Synchronise clocks with a reliable time source

**Article**: RTS Art. 12(2)(f). **Basis**: Explicit objective. Practice for the AWS measures.

- EC2 instances and AMIs use the Amazon Time Sync Service (`169.254.169.123`) in the chrony configuration.
- Record the time source in the repository documentation.

## LOG-08: Design logs for incident reconstruction

**Article**: RTS Art. 12(2)(b) and 23(4); DORA Art. 13(2)(b); RTS 2024/1772 Art. 3. **Basis**: Explicit objective. Practice measure.

Logs must show the full timeline of an incident, not only the data for debugging. DORA Art. 13(2)(b) examines the quality and the speed of the forensic analysis after a major incident.

- Write application logs as structured JSON.
- Give each event a UTC timestamp, a request ID, the principal, the resource ARN, the action and the result.
- Send one trace ID through all services, for example with the `X-Amzn-Trace-Id` header or AWS X-Ray.
- Make the log archive searchable with Amazon Athena or CloudWatch Logs Insights. One query can then show the events from all accounts.

## DET-01: Detect anomalous activity

**Article**: DORA Art. 10(1); RTS Art. 23(2)(a) and (b). Simplified: RTS Art. 34(g). **Basis**: Explicit objective. Practice for the AWS measures. RTS Art. 23(2)(b) asks for alert tools at minimum for CIF assets.

- Turn on GuardDuty in all accounts and Regions through a delegated administrator account.
- Turn on the GuardDuty protection plans that agree with the workload: S3, EKS, RDS, Lambda, Malware Protection and Runtime Monitoring.
- Turn on Security Hub and aggregate the findings in one Region.
- Add CloudWatch anomaly detection alarms on the key metrics of CIF workloads.

## DET-02: Send alerts to the response staff at all hours

**Article**: DORA Art. 10(2); RTS Art. 23(2)(c). **Basis**: Explicit objective. Practice for the AWS measures.

- Send GuardDuty, Security Hub and alarm events through EventBridge to an SNS topic.
- Connect the SNS topic to the on-call tool, for example AWS Systems Manager Incident Manager.
- Set the alert priority from the finding severity and the `dora:cif` tag.

The initial notification of a major incident is due as early as possible, within 4 hours after classification as major and no later than 24 hours after awareness (RTS 2025/301 Art. 5(1)(a)). If classification as major occurs after the first 24 hours, Art. 5(2) requires notification within 4 hours after classification. The entity must also assess the calendar provisions and exceptions in Art. 5(4) to (6). Alerts must arrive in time for the applicable deadline. See [Regulation 2025/301, Article 5](https://eur-lex.europa.eu/eli/reg_del/2025/301/oj/eng).

## DET-03: Record the time of occurrence, the time of detection and the type

**Article**: RTS Art. 23(4); RTS 2024/1772 Art. 3(1). **Basis**: Explicit objective. Practice for the AWS measures.

RTS 2024/1772 Art. 3(1) measures the incident duration from the time of occurrence. The entity finds this time in the logs.

- Keep GuardDuty and Security Hub findings in the log archive. They contain the first observed time, the creation time and the type.
- Keep logs long enough to find the start of an incident (LOG-05).

## DET-04: Test the detection and the alert path

**Article**: DORA Art. 10(1), second subparagraph. **Basis**: Explicit objective. Practice for the AWS measures.

A documented control does not show that it works in an incident.

- Add a scheduled job that creates GuardDuty sample findings with `CreateSampleFindings`.
- Make sure that each sample finding gets to the on-call tool (DET-02). Mark it as a test.
- Add a scheduled test of the logging failure alert (LOG-06).
- Record the result of each test in the log archive.

## Out of IaC scope

- The list of events to log and the purpose of each log (RTS Art. 12(2)(a)).
- The incident management policy and the contact list (RTS Art. 22).
- The roles and responsibilities for detection and response (RTS Art. 23(1)).
- The test plan for the detection mechanisms and the review of the test results (DORA Art. 10(1)).
- The classification and the reports of major incidents (DORA Art. 18 and 19).
