---
name: dora-aws-iac
description: DORA (Regulation (EU) 2022/2554) controls for AWS infrastructure as code. Use when you write or review Terraform, CDK or CloudFormation for an EU financial entity, or when the user mentions DORA.
---

# DORA controls for AWS IaC

Use this skill to write AWS infrastructure as code (IaC) that supports DORA compliance. You can also use it to examine IaC that is already in the repository.

## Terms

- **DORA**: Regulation (EU) 2022/2554.
- **RTS**: Commission Delegated Regulation (EU) 2024/1774. It gives the technical details for DORA Articles 9 to 16.
- **Simplified framework**: the rules for the small entities in DORA Art. 16(1). These entities use RTS Title III (Art. 28 to 41), not RTS Title II.
- **CIF**: a critical or important function. Many controls are stricter for a CIF.
- **Control**: one technical measure in the catalogue. Each control has an ID, for example `ENC-01`.
- **Evidence**: the IaC resource or attribute that shows a control.
- **Gap**: an applicable control that has no evidence.
- **Control map**: the table that records the evidence and the gaps for each resource.

## The limits of IaC evidence

DORA tells the entity to have policies, procedures and tools. IaC can show only the tools and their settings. IaC alone cannot make an entity compliant.

- Write that the IaC "supports" a DORA article. Do not write that the IaC "is DORA compliant".
- Use only the article numbers in the catalogue. For a control that is not in the catalogue, record the basis as Practice.
- Tell the user the basis of each control:
  - **Explicit**: the regulation text gives the measure or the number.
  - **Practice**: the regulation gives an objective. The AWS measure is a usual way to get that objective.

## Procedure

### 1. Get the context

Find these facts before you write IaC. Look in the repository first, for example in the README, ADRs, tags and variables. Ask the user for the facts that you cannot find.

| Fact | Effect on the IaC | Value if the user does not know |
|---|---|---|
| Entity type | Selects RTS Title II or the simplified framework. CCPs, CSDs and trading venues have stricter recovery limits (BCK-05). | RTS Title II |
| CIF | A CIF gets stricter controls and shorter intervals. | CIF |
| Data classification | Sets the encryption, access and backup scope. | Confidential |
| RTO and RPO | Set the backup frequency, the redundancy and the DR pattern. | Required input variable |
| Regions | The register of information records the country of data storage and processing (AST-03). | Required input variable |
| Retention periods | The entity sets log and backup retention from its risk assessment (LOG-05). | Required input variable |

If the user does not know a fact, use the value in the table. Record each such value as an assumption in the control map.

Step 1 is complete when each fact has a value or a recorded assumption.

### 2. Select the controls

Make a list of each AWS resource that you will write or examine. For each resource, read the catalogue files that this table gives for its type.

| Catalogue file | Read it when the IaC contains |
|---|---|
| [encryption.md](references/encryption.md) | data stores, queues, streams, KMS keys, certificates, TLS endpoints |
| [network.md](references/network.md) | VPCs, subnets, security groups, load balancers, gateways, VPC endpoints, CloudFront |
| [identity-access.md](references/identity-access.md) | IAM, IAM Identity Center, Cognito, SCPs, administrator access paths |
| [logging-detection.md](references/logging-detection.md) | accounts, CloudTrail, log destinations, alarms, CIF compute or data stores |
| [backup-recovery.md](references/backup-recovery.md) | data stores, AWS Backup, Multi-AZ or multi-Region designs, CIF workloads |
| [operations-change.md](references/operations-change.md) | accounts, pipelines, IaC state, compute, images, container registries, scaling |
| [assets-third-party.md](references/assets-third-party.md) | AWS Organizations, Region choice, AWS Config, connections to external services |

Step 2 is complete when each resource has a list of applicable control IDs.

### 3. Write the IaC

For a review task, go to step 4.

1. Add the DORA tags (AST-01) to each resource that accepts tags. Use provider default tags, `Tags.of()` in CDK, or stack tags.
2. Write the evidence for each applicable control.
3. Put entity-specific values in input variables. Examples are retention periods, RTO, RPO, Regions, CIDR ranges and key administrators.
4. Add a comment with the control ID next to each evidence, for example `# DORA ENC-01`.

#### AST-01: DORA tags

**Article**: RTS Art. 4(2)(b); DORA Art. 8(1). Simplified: RTS Art. 30(1). **Basis**: Explicit for the data. Practice for tags.

RTS Art. 4(2)(b) tells the entity to record these data for each ICT asset. AWS gives the unique identifier (ARN) and the location (Region).

| Tag key | Value | Article |
|---|---|---|
| `dora:function` | The business function or service that the resource supports | RTS Art. 4(2)(b)(v) |
| `dora:cif` | `true` or `false` | DORA Art. 8(1) |
| `dora:classification` | The data classification level | RTS Art. 4(2)(b)(iii) |
| `dora:owner` | The team that owns the asset | RTS Art. 4(2)(b)(iv) |
| `dora:rto`, `dora:rpo` | The recovery objectives, for example `4h` | RTS Art. 4(2)(b)(vi) |
| `dora:internet-exposed` | `true` or `false` | RTS Art. 4(2)(b)(vii) |

### 4. Examine each resource

For each resource, examine each applicable control. Give each pair one status:

- **Met**: give the evidence as `file:line`.
- **Gap**: tell what is missing and how to add it.
- **Out of IaC scope**: tell the policy or process that the entity must have.
- **Assumed**: the status depends on an assumption from step 1.

If the repository has a policy scanner (Checkov, cfn-guard, cdk-nag or Trivy), run it. Add its findings to the related controls.

In a write task, add the evidence for each Gap that IaC can close. Then examine that resource again.

Step 4 is complete when each pair of resource and applicable control has a status.

### 5. Write the control map

Write the control map to `docs/dora-control-map.md`, or to the path that the user gives. If the file exists, update it.

```markdown
| Control | Article | Basis | Resource | Status | Evidence or action |
|---|---|---|---|---|---|
| ENC-01 | DORA Art. 9(2); RTS Art. 6(2)(a) | Explicit | aws_s3_bucket.ledger | Met | storage.tf:42 |
| LOG-05 | RTS Art. 12(2)(a) | Explicit | aws_cloudwatch_log_group.app | Assumed | Retention is a required variable. The entity must set it. |
```

Under the table, add two lists:

1. **Assumptions**: each value from step 1 that the user did not give.
2. **Out of IaC scope**: each policy or process that the entity must have, with its article. Each catalogue file has a list of these items.

In your answer, give the Gaps first. Then give the path to the control map.
