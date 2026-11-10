# Operations and change controls

## OPS-01: Separate production from non-production

**Article**: RTS Art. 8(2)(b)(v) and 16(5). **Basis**: Explicit.

RTS Art. 8(2) includes accounts, data and connections in the separation.

- Use a separate AWS account for each environment.
- Add no peering or Transit Gateway routes between production and non-production.
- Use separate KMS keys for each environment (ENC-04).
- Non-production gets only anonymised, pseudonymised or random data. Write no copy job from production to non-production without this step.

## OPS-02: Deploy all changes through a controlled pipeline

**Article**: DORA Art. 9(4)(e); RTS Art. 16(1)(c) and 17(1). Simplified: RTS Art. 38(2). **Basis**: Explicit for the control. Practice for the pipeline.

- Keep all IaC in version control.
- The pipeline shows the plan or the change set, and runs the policy scanner.
- A person who did not write the change approves it (RTS Art. 17(1)(b)).
- Only the pipeline can use the deploy role. Limit the OIDC trust policy to the repository and the protected branch.
- Add a rollback path: CloudFormation rollback triggers, CodeDeploy automatic rollback, or the previous Terraform state version.
- Keep Terraform state in an encrypted S3 bucket with versioning and state lock.

## OPS-03: Use a secure configuration baseline and examine it regularly

**Article**: RTS Art. 11(2)(b), 11(2)(k) and 13(k). **Basis**: Explicit for the baseline. Practice for the AWS measures.

- Turn on Security Hub with the AWS Foundational Security Best Practices standard. This standard contains the vendor settings in RTS Art. 11(2)(k).
- Add AWS Config conformance packs for the baseline.
- Build hardened AMIs with EC2 Image Builder.
- Set IMDSv2 to required on all EC2 instances and launch templates.

## OPS-04: Detect drift from the IaC

**Article**: RTS Art. 11(2)(b); DORA Art. 9(4)(e). **Basis**: Practice.

- Run CloudFormation drift detection or a `terraform plan` on a schedule.
- Send an alert when the result shows a difference (DET-02).

## OPS-05: Scan for vulnerabilities automatically

**Article**: RTS Art. 10(2)(b) and 10(2)(d). Simplified: RTS Art. 34(d). **Basis**: Explicit. RTS Art. 10(2) asks for scans at least each week for CIF assets.

- Turn on Amazon Inspector for EC2, ECR and Lambda. Inspector scans continuously.
- Turn on ECR enhanced scanning.
- Scan third-party and open-source libraries in the pipeline.

## OPS-06: Patch with deadlines

**Article**: DORA Art. 9(4)(f); RTS Art. 10(4). Simplified: RTS Art. 34(d). **Basis**: Explicit.

- Use SSM Patch Manager with patch baselines and maintenance windows.
- Put the approval delay for patches in a variable.
- Turn on automatic minor version upgrades for managed databases.

## OPS-07: Plan capacity and prevent shortages

**Article**: DORA Art. 7(c) and 7(d); RTS Art. 9(1). Simplified: RTS Art. 34(c). **Basis**: Explicit.

- Use Auto Scaling with limits from load tests.
- Add CloudWatch alarms on utilisation for CIF resources.
- Add Service Quotas alarms for the quotas that a CIF workload uses.
- For Lambda functions of a CIF, set reserved concurrency.

## OPS-08: Allow only authorised software and block malicious code

**Article**: RTS Art. 11(2)(c), 11(2)(d) and 16(7). Simplified: RTS Art. 34(i). **Basis**: Explicit objective. Practice measure.

- Pull container images only from approved ECR repositories.
- Sign images and Lambda code with AWS Signer. Make signatures necessary at deployment.
- Turn on GuardDuty Malware Protection (DET-01).

## OPS-09: Prevent data leakage and delete data that is not necessary

**Article**: RTS Art. 11(2)(g), 11(2)(i) and 14(1)(b). Simplified: RTS Art. 35(e). **Basis**: Explicit objective. Practice measure.

- Turn on Amazon Macie for buckets with Confidential data.
- Add VPC endpoint policies that allow access only to buckets in the organisation (`aws:ResourceOrgID`).
- Add S3 lifecycle expiration rules for data that the entity no longer needs.

## OPS-10: Monitor each control continuously

**Article**: DORA Art. 9(1); RTS Art. 2(2)(c)(i) and 11(2)(b). **Basis**: Explicit for the monitoring. Practice for the AWS measures.

Controls decay silently after deployment.

- For each Met control, find the AWS Config rule or the Security Hub control that examines the evidence.
- If AWS has such a rule or control, add it.
- Send an alert when a rule or a control becomes non-compliant (DET-02).
- In the control map, add the rule name or the control ID to the evidence.

## OPS-11: Make the build chain verifiable

**Article**: RTS Art. 10(2)(d), 16(7) and 16(8). **Basis**: Explicit objective. Practice measure.

An unsigned pipeline and unknown sub-suppliers keep SOV-5 at SEAL-1 (AST-09).

- Make a software bill of materials (SBOM) for each build. Include the licence of each component.
- Stop the deployment if the SBOM is missing or contains a critical vulnerability.
- Sign each artifact and its build provenance (OPS-08). Use SLSA Build Level 2 as the minimum.
- Keep the SBOMs and the provenance with each release, so that an auditor can read them.

## Out of IaC scope

- The ICT operations policies and procedures (RTS Art. 8).
- The capacity and performance management procedures (RTS Art. 9).
- The vulnerability disclosure procedure (RTS Art. 10(2)(e)).
- The change management procedure and the procedure for emergency changes (RTS Art. 17(1)(f) and (g)).
- The source code review and the security tests before deployment (RTS Art. 16(3) and 16(4)).
- The ICT project management policy (RTS Art. 15).
