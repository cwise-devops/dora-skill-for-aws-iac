# Asset and third-party controls

AST-01 (DORA tags) is in `SKILL.md`, step 3.

## AST-02: Keep an automatic inventory of assets and their configuration

**Article**: DORA Art. 8(4) and 8(6); RTS Art. 4(2)(a). **Basis**: Explicit for the inventory. Practice for the AWS measures.

- Turn on the AWS Config recorder in all accounts and Regions (LOG-03).
- Add an AWS Config aggregator in the audit account.
- Add AWS Resource Groups that select resources by the `dora:function` tag.

## AST-03: Keep data in the approved Regions and record their countries

**Article**: ITS 2024/2956 Annex I, template B_02.02, fields B_02.02.0150 and B_02.02.0160; RTS Art. 26(2)(h); EBA/GL/2026/06 para. 214(j). **Basis**: Explicit for the record. Practice for the SCP.

The register of information records the country of the data at rest and the country of the data processing. It uses ISO 3166-1 alpha-2 codes. From 1 January 2027, supervisors of credit institutions also examine these locations in the SREP. They examine the regions and countries of data centres, data storage and data processing (EBA/GL/2026/06 para. 214(j)).

- Put the approved Regions in a variable.
- Add an SCP that denies requests outside the approved Regions with `aws:RequestedRegion`. Exempt the global services.
- Make sure that replication and backup copy destinations are in approved Regions.
- Add an output that gives the country code for each Region in use.

## AST-04: Map the supply chain behind each external service

**Article**: DORA Art. 8(5) and 28(3); ITS 2024/2956 Art. 2, Art. 3(2)(b) and template B_05.02. **Basis**: Explicit for the record. Practice for the IaC measures.

The register of information gives each provider a rank. The direct provider has rank 1. Its subcontractors have rank 2 or higher. For a CIF, the register must contain each subcontractor that effectively underpins the service.

A different provider on the same infrastructure is not a diversification. In the SREP, supervisors assess the concentration of CIFs on one provider, directly or through subcontracting chains (EBA/GL/2026/06 para. 194).

- Record each external service in `dora/dependencies.yaml`. Examples are SaaS APIs, AWS Marketplace products, PrivateLink services and Amazon Bedrock models.
- Make the IaC read this file for the egress allow list (NET-04). Then the IaC can connect only to recorded services.
- For each external service of a CIF, record the provider behind the provider, for example its hosting provider and Region.
- Set `shared_failure_domain: true` if the service runs on AWS, or in the same Region as the workload.

```yaml
- name: card-processor            # The same name as in the register of information
  provider: Example Payments SA
  functions: [card-payments]      # Values of dora:function
  endpoints: [api.example-pay.eu] # The egress allow list
  hosting: aws/eu-central-1       # The provider behind the provider, if known
  shared_failure_domain: true
  data_stored: true               # B_02.02.0140
  storage_countries: [DE]         # B_02.02.0150
  processing_countries: [DE]      # B_02.02.0160
```

## AST-05: Track end-of-support dates

**Article**: DORA Art. 8(7); RTS Art. 4(2)(b)(ix). Simplified: RTS Art. 34(e). **Basis**: Explicit for the record. Practice for the AWS measures.

- Put engine, runtime and cluster versions in variables.
- Send AWS Health lifecycle events through EventBridge to the alert topic (DET-02).

## AST-06: Decompose the exit strategy for each CIF component

**Article**: DORA Art. 28(8); ITS 2024/2956 template B_07.01, fields B_07.01.0050, B_07.01.0080 and B_07.01.0110. **Basis**: Explicit for the exit plan. Practice for the decomposition.

DORA Art. 28(8) tells the entity to move services and data to another provider, or back in-house, without disruption. An exit plan in a document does not show that the entity can replace a service in an acceptable time.

- For each CIF component, record the AWS service, its portability, the data export path and the alternative.
- Portability is Standard for open engines and APIs, for example PostgreSQL on RDS, Kubernetes on EKS or S3 objects.
- Portability is Proprietary for AWS-only APIs, for example DynamoDB or Step Functions.
- Suggest a substitutability value from field B_07.01.0050: not substitutable, highly complex, medium complexity, or easily substitutable. The entity confirms the value.
- Make sure that the alternative does not use the same infrastructure as the component (AST-04).
- Keep all infrastructure in IaC, so that the transition plan can use it.

## AST-07: Make the IaC a data source for the register of information

**Article**: DORA Art. 28(3); ITS 2024/2956 Art. 3(3). **Basis**: Explicit for accurate and consistent data. Practice for the IaC output.

Automate the data source, not the document.

- Add an IaC output in JSON. For each `dora:function`, give the AWS services, the Regions, the country codes and the external services from `dora/dependencies.yaml`.
- For each type of ICT service, give the providers that the CIFs use. Supervisors use the register to see the degree of concentration (EBA/GL/2026/06 para. 214(e)).
- Use the names from the controlled list (AST-01).

## AST-08: Find changes that affect the register of information

**Article**: DORA Art. 28(3), fifth subparagraph. **Basis**: Explicit.

The entity must tell the competent authority in time about each planned contractual arrangement for a CIF. It must also tell the authority when a function becomes critical or important.

Record each of these changes in the control map:

- A new external service for a CIF.
- A new Region, or a new country, for CIF data.
- A change of `dora:cif` from `false` to `true`.

## AST-09: Keep data, keys and dependencies inside the sovereignty target

**Article**: DORA has no article. RTS Art. 26(2)(h) tells the entity to plan for political instability in the provider's jurisdiction and in the data location. **Basis**: Guidance: European Commission Cloud Sovereignty Framework v1.2.1, as explained at <https://www.eucloudpatterns.eu/seal-framework/> and <https://www.eucloudpatterns.eu/seal-framework/two-architectures-seal-profiles/>.

The framework is a procurement benchmark, not EU law. The revised EBA SREP Guidelines (EBA/GL/2026/06) do not refer to SEAL and do not ask for EU data residency.

The framework gives one SEAL level for each of the eight sovereignty objectives (SOV-1 to SOV-8). Record the target and the result for each objective, not one label. IaC cannot set a SEAL level. IaC can keep data, keys and dependencies inside the target.

| Level | Name | Use |
|---|---|---|
| SEAL-0 | No sovereignty | Not suitable for DORA workloads |
| SEAL-1 | Jurisdictional sovereignty | The target for most fintech and banking workloads |
| SEAL-2 | Data sovereignty | An uplift on SOV-2 and SOV-3 with BYOK, if the entity asks for it |
| SEAL-3 | Digital resilience | Needs a change of provider for SOV-1 to SOV-6 |
| SEAL-4 | Full digital sovereignty | National security and defence |

An AWS workload in an EU Region is usually SEAL-1 overall. IaC has an effect on these objectives:

| Objective | IaC measure | Controls |
|---|---|---|
| SOV-2 Legal and jurisdictional | BYOK for regulated workloads | AST-09 |
| SOV-3 Data and AI | EU Regions and BYOK | AST-03, AST-09 |
| SOV-4 Operational | The full infrastructure in IaC, and portable services | AST-06 |
| SOV-5 Supply chain | Signed builds, an SBOM gate, and the provider behind each provider | OPS-11, AST-04 |
| SOV-6 Technology | Non-proprietary APIs and a licence inventory | AST-06, OPS-11 |
| SOV-7 Security and compliance | This catalogue | All |

For SEAL-1:

- Put only EU Regions in the Region variable and in the SCP (AST-03).
- Make sure that replication, backup copies and log archives stay in EU Regions.

For SEAL-2 on SOV-2 and SOV-3, if the entity asks for it:

- Use KMS keys with imported key material (BYOK) for regulated workloads. Generate the key material in an HSM in the EU, outside AWS.
- KMS automatic rotation (ENC-04) does not apply to imported key material. The entity rotates the key material and keeps its backup. Record this in the control map.
- BYOK does not stop a legal order to AWS. AWS KMS still holds a working copy of the key.

For SEAL-3 on SOV-3, hold your own key (HYOK) with an AWS KMS external key store. The only master key then stays in the entity's HSM outside AWS. SEAL-3 on SOV-1 to SOV-6 also needs a change of provider. If the target is SEAL-3, record a Gap for these objectives and tell the user.

## Out of IaC scope

- The register of information (DORA Art. 28(3); ITS 2024/2956).
- The notification to the competent authority (DORA Art. 28(3)).
- The contractual arrangements and the policy on ICT services for CIF (DORA Art. 28 to 30; RTS 2024/1773).
- The assessment of ICT concentration risk (DORA Art. 29).
- The exit plans and their tests (DORA Art. 28(8)).
- The SEAL assessment for each sovereignty objective.
- The countries from where the provider gives support, also for escalation outside working hours (ITS 2024/2956 field B_02.02.0130).
- The risk assessment of legacy ICT systems (DORA Art. 8(7)).
