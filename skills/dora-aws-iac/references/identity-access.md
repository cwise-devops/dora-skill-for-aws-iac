# Identity and access controls

## IAM-01: Give least privilege

**Article**: DORA Art. 9(4)(c); RTS Art. 21(a) and 21(d). Simplified: RTS Art. 33(a). **Basis**: Explicit.

- Policies name the actions and the resource ARNs. Use wildcards only where the API has no resource-level permissions.
- Add conditions, for example `aws:PrincipalOrgID`, `aws:ResourceOrgID` and `aws:SourceAccount`.
- Put permission boundaries on roles that pipelines or developers create.
- Use SCPs to deny high-risk actions in member accounts. Examples: leave the organisation, stop CloudTrail, GuardDuty or Config, or use a Region that is not approved (AST-03).
- Turn on IAM Access Analyzer for external access and unused access.

## IAM-02: Give each person and each system a unique identity

**Article**: RTS Art. 20(1), 20(2) and 21(c). Simplified: RTS Art. 33(b). **Basis**: Explicit.

- People sign in through IAM Identity Center with the corporate identity provider. Write IAM roles and permission sets, not IAM users, for people.
- Provision identities from the identity provider with SCIM. RTS Art. 20(2) asks for automated identity lifecycle where feasible.
- Systems use IAM roles. CI/CD uses OIDC federation instead of long-term access keys.
- Use centralised root access management in AWS Organizations to remove root credentials from member accounts.

## IAM-03: Use strong authentication

**Article**: DORA Art. 9(4)(d); RTS Art. 21(f). Simplified: RTS Art. 33(d). **Basis**: Explicit.

RTS Art. 21(f)(ii) asks for strong authentication for remote access, privileged access, access to CIF assets and access to publicly accessible assets.

- Make MFA necessary for all users in IAM Identity Center or in the identity provider.
- Cognito user pools for CIF or public applications: set MFA to on.
- Client VPN: use federated or certificate authentication with MFA.

## IAM-04: Give privileged access only when it is necessary

**Article**: RTS Art. 21(e)(ii). Simplified: RTS Art. 33(c). **Basis**: Explicit.

- Use dedicated permission sets or roles for administrator tasks.
- Give elevated access for a limited time and with approval, for example through a just-in-time access tool.
- Keep a break-glass role. Send an alert each time that someone uses it (DET-02).
- CloudTrail records all privileged actions (LOG-01).

## IAM-05: Separate duties

**Article**: RTS Art. 21(b) and 17(1)(b). **Basis**: Explicit.

- In production, people get read-only access. Only the pipeline role deploys changes (OPS-02).
- Key administrators cannot use keys to decrypt data (ENC-04).
- Workload administrators cannot change the log archive account or the backup account (LOG-04, BCK-02).

## IAM-06: Make access reviews possible

**Article**: RTS Art. 21(e)(iv). Simplified: RTS Art. 33(e). **Basis**: Explicit. The entity must review access every 6 months for a CIF, and every year for other systems.

- Turn on IAM Access Analyzer unused access findings.
- Keep all permission sets and role policies in IaC, so that a reviewer can read them.

## Out of IaC scope

- The access review, at the intervals in IAM-06 (RTS Art. 21(e)(iv)).
- The removal of access rights when employment ends (RTS Art. 21(e)(iii)).
- The records of all identity assignments (RTS Art. 20(2)).
- Physical access controls (RTS Art. 21(g)).
