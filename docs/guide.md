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

The skill starts automatically when you mention DORA, or when you write IaC for an EU financial entity. You can also start it by name:

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
- Sovereignty target: SEAL-1
- Function names (controlled list): card-payments, customer-onboarding, marketing-site
```

The values above are examples. Use the values of your entity. DORA gives no retention period. Your ICT risk assessment sets it.

If a fact is missing, the skill uses these defaults and records them as assumptions:

| Fact | Default |
|---|---|
| Entity type | RTS Title II (full framework) |
| CIF | The workload supports a CIF |
| Data classification | Confidential |
| RTO, RPO, Regions, retention | Required input variables with no default value |
| Sovereignty target | SEAL-1 |

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
Review the CDK app in infra/ against DORA. Do not change the code yet.
```

For a review, the skill skips step 3. It examines the code that is in the repository.

**Example answer (part):**

```text
Gaps (3):
- BCK-02 LedgerDb: backups stay in the workload account. Add a copy to the
  backup account with AWS Backup Vault Lock.
- NET-03 BastionSg: port 22 is open to 0.0.0.0/0. Use SSM Session Manager.
- LOG-05 ApiLogs: retention is 7 days and fixed in code. Use a required variable.

Register changes (1):
- AST-08: new external service api.example-pay.eu for card-payments.

Control map: docs/dora-control-map.md
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
| Small entity (DORA Art. 16(1)) | "Entity type: simplified framework" | The skill refers to RTS Title III articles |
| CCP, CSD or trading venue | The entity type | Recovery limits of 2 hours or less, and a secondary site in a second Region (BCK-05) |
| Stronger key control | "Sovereignty target: SEAL-2 on SOV-2 and SOV-3" | KMS keys with imported key material (BYOK) for regulated workloads (AST-09) |
| Workload that does not support a CIF | "marketing-site is not a CIF" | Fewer controls, and longer review intervals |

## 8. Read the control map

Each row is one pair of resource and control:

| Control | Article | Basis | Resource | Status | Evidence or action |
|---|---|---|---|---|---|
| ENC-01 | DORA Art. 9(2); RTS Art. 6(2)(a) | Explicit | aws_rds_cluster.ledger | Met | database.tf:14 |
| BCK-02 | DORA Art. 12(3) | Explicit | aws_rds_cluster.ledger | Gap | Add a cross-account backup copy |
| LOG-05 | RTS Art. 12(2)(a) | Explicit | aws_cloudwatch_log_group.ledger | Assumed | The entity must set the retention |

| Status | Meaning |
|---|---|
| Met | The IaC shows the control. The evidence gives the file and the line. |
| Gap | The control has no evidence. The action tells how to add it. |
| Out of IaC scope | The control needs a policy or a process. IaC cannot show it. |
| Assumed | The status depends on a default value from step 1. |

| Basis | Meaning |
|---|---|
| Explicit | The regulation text gives the measure or the number. |
| Practice | The regulation gives an objective. The AWS measure is a usual way to get it. |
| Guidance | A framework or a supervisor gives the measure, for example the SEAL framework. |

Under the table, the control map has four lists:

1. **Assumptions**: the default values that the skill used.
2. **Out of IaC scope**: the policies and processes that your entity must have.
3. **Exit decomposition**: for each CIF component, the portability, the substitutability and the alternative.
4. **Register changes**: the changes that can affect the register of information.

## 9. Limits

- IaC shows only the technical part of DORA. The skill does not make an entity compliant. It writes that the IaC "supports" an article.
- Give the control map to your compliance team. They own the policies, the register of information and the notifications.
- The skill cites only the articles in its catalogue. Examine important claims against the regulation text.
