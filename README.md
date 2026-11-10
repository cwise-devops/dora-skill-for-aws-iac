# dora-skill-for-aws-iac

An agent skill for writing and reviewing AWS infrastructure as code (Terraform, CDK, CloudFormation) that supports DORA compliance. The skill is written in ASD-STE100 Simplified Technical English.

## Layout

- `skills/dora-aws-iac/SKILL.md`: the procedure (context, controls, write, examine, control map) and the DORA tag set.
- `skills/dora-aws-iac/references/`: the control catalogue, with one file for each domain. Each control gives the articles, the basis (Explicit, Practice or Guidance) and the AWS evidence.
- `docs/guide.md`: how to use the skill, with examples.

## Install

Copy `skills/dora-aws-iac` into `~/.claude/skills/` (personal) or `.claude/skills/` (project).

## Usage

Mention DORA, or type `/dora-aws-iac`. Read [docs/guide.md](docs/guide.md) for the setup, example prompts and how to read the control map.

## Sources

- Regulation (EU) 2022/2554 (DORA)
- Delegated Regulation (EU) 2024/1774: ICT risk management RTS and the simplified framework
- Delegated Regulation (EU) 2024/1772: incident classification RTS
- Delegated Regulation (EU) 2025/301: incident report time limits
- Implementing Regulation (EU) 2024/2956: register of information ITS

Guidance (controls with the basis "Guidance"):

- European Commission, Cloud Sovereignty Framework v1.2.1 (20 October 2025), as explained at <https://www.eucloudpatterns.eu/seal-framework/> and <https://www.eucloudpatterns.eu/seal-framework/two-architectures-seal-profiles/>
- EBA, Guidelines on SREP and supervisory stress testing (EBA/GL/2026/06), apply from 1 January 2027: <https://www.eba.europa.eu/sites/default/files/2026-06/fd5fbfa1-2efb-4122-8e91-4831469d8150/Final%20Report%20on%20revised%20SREP%20and%20supervisory%20stress%20testing%20Guidelines.pdf>
