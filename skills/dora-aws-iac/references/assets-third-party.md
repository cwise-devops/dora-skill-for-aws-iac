# Asset and third-party controls

AST-01 (DORA tags) is in `SKILL.md`, step 3.

## AST-02: Keep an automatic inventory of assets and their configuration

**Article**: DORA Art. 8(4) and 8(6); RTS Art. 4(2)(a). **Basis**: Explicit for the inventory. Practice for the AWS measures.

- Turn on the AWS Config recorder in all accounts and Regions (LOG-03).
- Add an AWS Config aggregator in the audit account.
- Add AWS Resource Groups that select resources by the `dora:function` tag.

## AST-03: Keep data in the approved Regions and record their countries

**Article**: ITS 2024/2956 Annex I, template B_02.02, fields B_02.02.0150 and B_02.02.0160; RTS Art. 26(2)(h). **Basis**: Explicit for the record. Practice for the SCP.

The register of information records the country of the data at rest and the country of the data processing. It uses ISO 3166-1 alpha-2 codes.

- Put the approved Regions in a variable.
- Add an SCP that denies requests outside the approved Regions with `aws:RequestedRegion`. Exempt the global services.
- Make sure that replication and backup copy destinations are in approved Regions.
- Add an output that gives the country code for each Region in use.

## AST-04: Record dependencies on third-party services

**Article**: DORA Art. 8(5) and 28(3). **Basis**: Explicit.

- Define each connection to an external service in IaC. Examples are SaaS endpoints, AWS Marketplace products, PrivateLink services and partner APIs.
- Add the DORA tags (AST-01) to each such resource.

## AST-05: Track end-of-support dates

**Article**: DORA Art. 8(7); RTS Art. 4(2)(b)(ix). Simplified: RTS Art. 34(e). **Basis**: Explicit for the record. Practice for the AWS measures.

- Put engine, runtime and cluster versions in variables.
- Send AWS Health lifecycle events through EventBridge to the alert topic (DET-02).

## AST-06: Support the exit strategy for CIF services

**Article**: DORA Art. 28(8). **Basis**: Explicit for the exit plan. Practice for the IaC measures.

DORA Art. 28(8) tells the entity to move services and data to another provider, or back in-house, without disruption.

- Keep data in formats that the entity can export, for example S3 objects or standard database engines.
- Keep all infrastructure in IaC, so that the transition plan can use it.

## Out of IaC scope

- The register of information (DORA Art. 28(3); ITS 2024/2956).
- The contractual arrangements and the policy on ICT services for CIF (DORA Art. 28 to 30; RTS 2024/1773).
- The assessment of ICT concentration risk (DORA Art. 29).
- The exit plans and their tests (DORA Art. 28(8)).
- The risk assessment of legacy ICT systems (DORA Art. 8(7)).
