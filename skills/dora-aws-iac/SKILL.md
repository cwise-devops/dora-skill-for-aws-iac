---
name: dora-aws-iac
description: DORA (Regulation (EU) 2022/2554) controls for AWS infrastructure as code. Use when you write or review Terraform, CDK or CloudFormation for an EU financial entity, or when the user asks to apply DORA controls to AWS IaC.
---

# DORA controls for AWS IaC

Use this skill to write AWS infrastructure as code (IaC) that supports DORA compliance. You can also use it to examine IaC that is already in the repository.

## Terms

- **DORA**: Regulation (EU) 2022/2554.
- **RTS**: Commission Delegated Regulation (EU) 2024/1774. It gives the technical details for DORA Articles 9 to 16.
- **Simplified framework**: the rules for the entity categories listed in DORA Art. 16(1). Size alone does not determine eligibility. These entities use RTS Title III (Art. 28 to 41), not RTS Title II.
- **CIF**: a critical or important function. Many controls are stricter for a CIF.
- **Control**: one technical measure in the catalogue. Each control has an ID, for example `ENC-01`.
- **Evidence**: the IaC resource or attribute that shows a control.
- **Gap**: an applicable control that the evidence shows is missing or inadequate.
- **Unknown**: an applicable control whose implementation or required context cannot be verified.
- **Control map**: the table that records the evidence and the gaps for each resource.

## The limits of IaC evidence

DORA tells the entity to have policies, procedures and tools. IaC can show only the tools and their settings. IaC alone cannot make an entity compliant.

- Write that the IaC "supports" a DORA article. Do not write that the IaC "is DORA compliant".
- Use only the article numbers in the catalogue. For a control that is not in the catalogue, record the basis as Practice.
- Tell the user the basis of each control:
  - **Explicit**: the regulation text gives the measure or the number.
  - **Practice**: the regulation gives an objective. The AWS measure is a usual way to get that objective.
  - **Guidance**: the regulation does not give the measure. A supervisor or a framework gives it. Tell the user the source.

## Task boundaries

For a review, inspect the code and report findings. Respect requests to make no file changes: return the control map in the answer instead of writing it, and do not create ADRs. Writing IaC does not authorise deployment, changes to live AWS resources, or changes to Terraform state.

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
| Sovereignty target, if requested | Adds entity-approved location, key custody and dependency requirements (AST-09). | No additional sovereignty profile |

If the user does not know a fact, use the value in the table. Record each such value as an assumption in the control map.

Required input variables are placeholders, not confirmed values. If RTO, RPO or approved Regions are unknown, you can write scaffolding, but leave the recovery architecture and Region selection unresolved. Do not select a DR pattern or report its adequacy until the objectives are known.

Step 1 is complete when each fact has a value, a recorded assumption or an unresolved input. Record unresolved inputs as Unknown in the control map.

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
| [assets-third-party.md](references/assets-third-party.md) | AWS Organizations, Region choice, AWS Config, external services, `dora/dependencies.yaml` |

Select the legal framework before assessing controls:

- Confirm eligibility for the simplified framework against DORA Art. 16(1). Do not infer eligibility from size or from microenterprise status.
- For the full framework, use the main article citations. Apply each stated microenterprise exception separately.
- For the simplified framework, use the Simplified citations. Do not carry over full-framework intervals, limits or obligations unless the simplified provision also requires them.
- A control without a Simplified citation is not automatically a legal requirement for a simplified-framework entity. Record it as optional Practice, or leave legal applicability Unknown pending a verified applicable provision. DORA obligations outside the ICT risk management framework, such as third-party obligations, require their own applicability assessment.
- Keep entity-approved implementation choices separate from legal requirements. An Explicit objective does not make a particular AWS service or topology mandatory.

Step 2 is complete when each resource has a list of applicable control IDs and the applicable framework and exceptions are recorded.

### 3. Write the IaC

For a review task, go to step 4.

1. Add the DORA tags (AST-01) to each resource that accepts tags. Use provider default tags, `Tags.of()` in CDK, or stack tags.
2. Write the evidence for each applicable control.
3. Make entity-specific values configurable. Examples are retention periods, RTO, RPO, Regions, CIDR ranges and key administrators. Use input variables for reusable modules or unresolved values. Documented, entity-approved constants are also valid.
4. Add a comment with the control ID next to each evidence, for example `# DORA ENC-01`.

#### AST-01: DORA tags

**Article**: RTS Art. 4(2)(b); DORA Art. 8(1); ITS 2024/2956 Art. 3(3). Simplified: RTS Art. 30(1). **Basis**: Explicit for the data. Practice for tags.

RTS Art. 4(2)(b) tells the entity to record these data for each ICT asset. AWS gives the unique identifier (ARN) and the location (Region).

Use one name for each function and each component. Use the same names in the tags, the register of information and the ADRs.

- Put the permitted values of `dora:function` and `dora:owner` in one list.
- Make the IaC reject a tag value that is not in the list, for example with a Terraform variable validation.

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

- **Met**: give the evidence as `file:line` and state what it proves.
- **Gap**: give evidence that the control is missing or inadequate, and tell how to correct it.
- **Unknown**: state the missing evidence or input and how to verify it. Absence from this repository alone does not prove that an account-level or central control is missing.
- **Out of IaC scope**: tell the policy or process that the entity must have.
- **Assumed**: the status depends on an assumption from step 1.

For each finding, record the evidence scope: source code, plan or synthesised template, deployed configuration, or operational test. Source code can show a configured measure, but does not prove deployment or operational effectiveness. Check references to central infrastructure before recommending duplicate resources. If central evidence is unavailable, use Unknown.

If the repository has a policy scanner (Checkov, cfn-guard, cdk-nag or Trivy), run it. Add its findings to the related controls.

In a write task, add the evidence for each Gap that IaC can close. Then examine that resource again.

Step 4 is complete when each pair of resource and applicable control has a status.

### 5. Write the control map

When file changes are allowed, write the control map to `docs/dora-control-map.md`, or to the path that the user gives. If the file exists, update it. For a review with no file changes, include the map in the answer.

```markdown
| Control | Article | Basis | Resource | Status | Evidence scope | Evidence or action |
|---|---|---|---|---|---|---|
| ENC-01 | DORA Art. 9(2); RTS Art. 6(2)(a) | Explicit objective; Practice measure | aws_s3_bucket.ledger | Met | Source code | storage.tf:42 configures encryption; deployment not verified. |
| LOG-05 | RTS Art. 12(2)(a) | Explicit objective; Practice measure | aws_cloudwatch_log_group.app | Unknown | Source code | Retention is a required variable. Verify its value against the approved policy. |
```

Under the table, add these lists:

1. **Assumptions and unresolved inputs**: each assumed value and missing input from step 1, plus the framework and exceptions used.
2. **Out of IaC scope**: each policy or process that the entity must have, with its article. Each catalogue file has a list of these items.
3. **Exit decomposition**: for each CIF component, the AWS service, its portability, its substitutability, the data export path and the alternative (AST-06).
4. **Register changes**: each change that can need an update of the register of information or a notification to the competent authority (AST-08).

If the repository keeps ADRs, write an ADR for each decision that sets a control. Examples are the DR pattern, the Regions and an accepted Gap.

In your answer, give the Gaps first, followed by Unknown findings. Then give the register changes and the path to the control map, or the inline map for a review with no file changes.
