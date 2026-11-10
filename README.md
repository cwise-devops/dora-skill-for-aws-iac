# dora-skill-for-aws-iac

An agent skill for writing and reviewing AWS infrastructure as code (Terraform, CDK, CloudFormation) that supports DORA compliance. The skill is written in ASD-STE100 Simplified Technical English.

## Layout

- `skills/dora-aws-iac/SKILL.md`: the procedure (context, controls, write, examine, control map) and the DORA tag set.
- `skills/dora-aws-iac/references/`: the control catalogue, with one file for each domain. Each control gives the articles, the basis (Explicit or Practice) and the AWS evidence.

## Install

Copy `skills/dora-aws-iac` into `~/.claude/skills/` (personal) or `.claude/skills/` (project).

## Sources

- Regulation (EU) 2022/2554 (DORA)
- Delegated Regulation (EU) 2024/1774: ICT risk management RTS, including the simplified framework
- Delegated Regulation (EU) 2024/1772: incident classification RTS
- Delegated Regulation (EU) 2025/301: incident report time limits
- Implementing Regulation (EU) 2024/2956: register of information ITS
