# Guide: use the dora-aws-iac skill

This guide shows how to use the skill to write and review AWS IaC for DORA. The examples use Terraform and CDK. The procedure is the same for CloudFormation.

## 1. Install the skill

Copy the skill folder to one of these locations:

| Scope | Location |
|---|---|
| All your projects | `~/.claude/skills/dora-aws-iac/` |
| One IaC repository | `<repository>/.claude/skills/dora-aws-iac/` |

```bash
git clone https://github.com/cwise-devops/dora-skill-for-aws-iac.git
cp -r dora-skill-for-aws-iac/skills/dora-aws-iac ~/.claude/skills/
```

Start a new Claude Code session after you install the skill.

## 2. Start the skill

The skill can start automatically when you ask to apply DORA controls to AWS IaC, or when you write or review AWS IaC for an EU financial entity. You can also start it by name:

```text
/dora-aws-iac
```

## 3. Give the context one time

In step 1, the skill looks for seven facts. If it cannot find a fact, it asks you or uses a strict default. Add a "DORA context" section to the README of your IaC repository. Then the skill finds the facts in each session.

```markdown
## DORA context

- Entity type: payment institution (RTS Title II)
- Critical or important functions: card-payments (CIF), customer-onboarding (CIF), marketing-site (not CIF)
- Data classification: card-payments is Confidential
- RTO / RPO: card-payments 4h / 15m
- Regions: eu-central-1 (DE), eu-west-1 (IE)
- Retention: <log and backup periods from your ICT risk assessment>
- Sovereignty profile: none requested; apply the approved Regions above
- Function names (controlled list): card-payments, customer-onboarding, marketing-site
```

The values above are examples. Use the values of your entity. DORA gives no retention period. Your ICT risk assessment sets it. A documented, approved retention constant can satisfy the control; an input variable alone does not establish adequacy.

Unknown RTO, RPO or approved Regions permit scaffolding only. The skill leaves the recovery architecture and Region selection unresolved until the values are available. Simplified-framework controls use their own citations and intervals; full-framework controls without a verified simplified basis are optional Practice or Unknown, rather than automatic legal obligations.

If a fact is missing, the skill uses these defaults and records them as assumptions:

| Fact | Default |
|---|---|
| Entity type | RTS Title II (full framework) |
| CIF | The workload supports a CIF |
| Data classification | Confidential |
| RTO, RPO, Regions, retention | Required input variables with no default value |
| Sovereignty target | No additional profile unless requested and approved by the entity |

## 4. Example: write new infrastructure

**Prompt:**

```text
Write Terraform for the card-payments ledger: an Aurora PostgreSQL cluster,
an S3 bucket for statements, and an ECS service behind an internal ALB.
```

**What the skill does:**

1. It reads the DORA context from the README.
2. It selects the controls for each resource. For this prompt, it reads the encryption, network, identity, logging, backup and operations files.
3. It writes the Terraform with the DORA tags and the evidence for each control.
4. It examines each resource against each control.
5. It writes `docs/dora-control-map.md`.

**Example result (part):**

```hcl
provider "aws" {
  region = var.region

  default_tags {
    tags = {
      "dora:function"         = var.dora_function # DORA AST-01
      "dora:cif"              = "true"
      "dora:classification"   = "confidential"
      "dora:owner"            = "payments-platform"
      "dora:rto"              = "4h"
      "dora:rpo"              = "15m"
      "dora:internet-exposed" = "false"
    }
  }
}

variable "dora_function" {
  type = string
  validation {
    condition     = contains(["card-payments", "customer-onboarding", "marketing-site"], var.dora_function)
    error_message = "Use a function name from the controlled list (DORA AST-01)."
  }
}

variable "log_retention_days" {
  type        = number
  description = "Log retention from the ICT risk assessment (DORA LOG-05). No default value."
}

resource "aws_rds_cluster" "ledger" {
  engine              = "aurora-postgresql"
  storage_encrypted   = true                  # DORA ENC-01
  kms_key_id          = aws_kms_key.ledger.arn
  # ...
}

resource "aws_cloudwatch_log_group" "ledger" {
  name              = "/ecs/card-payments-ledger"
  retention_in_days = var.log_retention_days  # DORA LOG-05
  kms_key_id        = aws_kms_key.logs.arn    # DORA ENC-01
}
```

In CDK, the skill adds the tags with `Tags.of(app).add("dora:function", "card-payments")`.

## 5. Example: review existing IaC

**Prompt:**

```text
Review the CDK app in infra/ against DORA. Do not change any files.
```

For a review, the skill skips step 3. It examines the code that is in the repository. If you request no file changes, it returns the control map in the answer and does not write files or ADRs.

**Example answer (part):**

```text
Gaps (3):
- BCK-02 LedgerDb: verified backups stay in the workload account, contrary to
  the approved policy. Add a copy to the backup account and select vault
  protection from that policy. Approve any immutable lock before deployment.
- NET-03 BastionSg: port 22 is open to 0.0.0.0/0. Use SSM Session Manager.
- LOG-05 ApiLogs: retention is 7 days, but the approved policy requires 90 days.
  Set the configured retention to the approved period.

Unknown (1):
- DET-01: central GuardDuty configuration is outside this repository. Verify it
  before recommending another detector.

Register changes (1):
- AST-08: new external service api.example-pay.eu for card-payments.

Control map: included in this answer because file changes were excluded.
```

Then ask the skill to close the gaps:

```text
Close the gaps that IaC can close. Update the control map.
```

## 6. Example: add an external service

**Prompt:**

```text
The card-payments service must call the Example Payments API at api.example-pay.eu.
```

The skill:

- Adds the service to `dora/dependencies.yaml` (AST-04).
- Builds the egress allow list from that file (NET-04).
- Records the change as a register change (AST-08). Your compliance team decides if the register of information needs an update, or if the competent authority needs a notification.

```yaml
- name: card-processor
  provider: Example Payments SA
  functions: [card-payments]
  endpoints: [api.example-pay.eu]
  hosting: aws/eu-central-1
  shared_failure_domain: true
  data_stored: true
  storage_countries: [DE]
  processing_countries: [DE]
```

## 7. Special cases

| Case | Tell the skill | Effect |
|---|---|---|
| Entity eligible under DORA Art. 16(1) | Give the entity category and confirmed eligibility | The skill uses RTS Title III citations; size alone does not establish eligibility |
| CCP, CSD or trading venue | The entity type | Recovery limits of 2 hours or less, and a secondary site in a second Region (BCK-05) |
| Stronger key control | Give the approved key custody and sovereignty requirements | The skill selects supported key controls and records evidence for a separate assessment; it does not assign a SEAL level (AST-09) |
| Workload that does not support a CIF | "marketing-site is not a CIF" | Fewer controls, and longer review intervals |

## 8. Read the control map

Each row is one pair of resource and control:

| Control | Article | Basis | Resource | Status | Evidence scope | Evidence or action |
|---|---|---|---|---|---|---|
| ENC-01 | DORA Art. 9(2); RTS Art. 6(2)(a) | Explicit objective; Practice measure | aws_rds_cluster.ledger | Met | Source code | database.tf:14 configures encryption; deployment not verified |
| BCK-02 | DORA Art. 12(2), 12(3) | Explicit objective; Practice measure | aws_rds_cluster.ledger | Gap | Deployed configuration | backup-review.md:12 confirms the only copy is in the workload account, contrary to the approved policy |
| LOG-05 | RTS Art. 12(2)(a) | Explicit objective; Practice measure | aws_cloudwatch_log_group.ledger | Unknown | Source code | The approved retention policy and effective input value are unavailable |

| Status | Meaning |
|---|---|
| Met | The evidence shows the measure within its stated scope. Source evidence does not prove deployment or operational effectiveness. |
| Gap | Evidence shows that an applicable control is missing or inadequate. The action tells how to correct it. |
| Unknown | Evidence, required context or central configuration is unavailable. The action tells how to verify it. |
| Out of IaC scope | The control needs a policy or a process. IaC cannot show it. |
| Assumed | The status depends on a default value from step 1. |

| Basis | Meaning |
|---|---|
| Explicit | The regulation text gives the measure or the number. |
| Practice | The regulation gives an objective. The AWS measure is a usual way to get it. |
| Guidance | A framework or a supervisor gives the measure, for example the SEAL framework. |

Under the table, the control map has four lists:

1. **Assumptions and unresolved inputs**: the defaults, missing inputs, selected framework and applicable exceptions.
2. **Out of IaC scope**: the policies and processes that your entity must have.
3. **Exit decomposition**: for each CIF component, the portability, the substitutability and the alternative.
4. **Register changes**: the changes that can affect the register of information.

## 9. Limits

- IaC shows only the technical part of DORA. The skill does not make an entity compliant. It writes that the IaC "supports" an article.
- Give the control map to your compliance team. They own the policies, the register of information and the notifications.
- The skill cites only the articles in its catalogue. Examine important claims against the regulation text.
