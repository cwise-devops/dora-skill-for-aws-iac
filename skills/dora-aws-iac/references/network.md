# Network controls

## NET-01: Segment networks by function and classification

**Article**: DORA Art. 9(4)(b); RTS Art. 13(a). **Basis**: Explicit objective. Practice for the AWS measures.

- Select VPC and account boundaries from the classification, risk assessment and dependencies. A separate VPC or account for each CIF workload is one implementation option.
- Use subnet tiers: public for load balancers only, private for compute, isolated for data stores.
- Isolated subnets have no route to the internet.
- Security group rules refer to other security groups, not to wide CIDR ranges.
- Remove all rules from the default security group.

## NET-02: Make each segment easy to isolate at once

**Article**: DORA Art. 9(4), second subparagraph; RTS Art. 13(j). **Basis**: Explicit objective. Practice for the AWS measures.

DORA tells the entity to design the network so that it can sever or segment a connection instantly.

- Write an isolation switch in IaC. For example, a variable removes the routes of a segment from the Transit Gateway or the internet gateway.
- Keep a quarantine security group with no rules. An incident runbook can attach it to an instance.
- Add an SSM Automation document that isolates an EC2 instance.

## NET-03: Use a separate path for administration

**Article**: RTS Art. 13(c). **Basis**: Explicit objective. Practice for the AWS measures.

- Use SSM Session Manager for shell access, through the VPC interface endpoints `ssm`, `ssmmessages` and `ec2messages`.
- Keep ports 22 and 3389 closed to the internet.
- Make EKS API endpoints private, or limit their public CIDR ranges.
- If bastion hosts are necessary, put them in a dedicated administration subnet or VPC.

## NET-04: Protect traffic between internal networks and the internet

**Article**: RTS Art. 13(g) and 14(1)(b). Simplified: RTS Art. 35(c). **Basis**: Explicit objective. Practice measure.

- Ingress: use CloudFront or an internet-facing load balancer with AWS WAF managed rule groups.
- Assess DDoS protection for CIF endpoints. Use AWS Shield Advanced if the risk assessment and response needs justify it; record the selection and cost as an implementation decision.
- Egress: use NAT gateways with AWS Network Firewall, or a proxy with a domain allow list. Build the allow list from `dora/dependencies.yaml` (AST-04).
- Use VPC endpoints for AWS services. Use gateway endpoints for S3 and DynamoDB. Limit each endpoint with an endpoint policy.
- Give instances no public IP addresses. Set RDS instances to not publicly accessible.
- Turn on S3 Block Public Access at the account level.

## NET-05: Prevent and detect unauthorised connections

**Article**: RTS Art. 13(d). Simplified: RTS Art. 35(c). **Basis**: Explicit objective. Practice for the AWS measures.

- Security groups and network ACLs allow only the necessary ports and sources.
- VPC Flow Logs (LOG-02) and GuardDuty (DET-01) detect unusual connections.
- For remote staff, use AWS Verified Access with device trust, or Client VPN. RTS Art. 13(d) includes endpoints that do not meet the security requirements.

## NET-06: Record each network connection and data flow

**Article**: RTS Art. 13(b); DORA Art. 8(4) and 8(5). **Basis**: Explicit objective. Practice for the AWS measures.

- Define all peering, Transit Gateway, VPN and Direct Connect connections in IaC.
- Give each security group rule a `description` that tells the data flow.
- Add the DORA tags (AST-01) to each connection to a third party.

## NET-07: Keep firewall rules in IaC for review

**Article**: RTS Art. 13(h). **Basis**: Explicit objective. Practice for the AWS measures. For a CIF, the entity must review the rules at least every 6 months.

- Keep all security group, network ACL, Network Firewall and WAF rules in IaC. The version history is then the change record.
- Use AWS Firewall Manager for rules that all accounts share.

## NET-08: End idle sessions

**Article**: RTS Art. 13(l). **Basis**: Explicit objective. Practice for the AWS measures.

- Session Manager: set `idleSessionTimeout` in the session preferences.
- Applications: implement an inactivity timeout that ends the session and requires authentication again. Verify the behaviour, including refresh tokens and session renewal.
- IAM Identity Center session duration, IAM role `max_session_duration` and Cognito token validity limit session or credential lifetime. They do not prove an inactivity timeout. Use them as additional controls.
- If a service cannot enforce inactivity termination, record a Gap and the proposed compensating measure. If the application or central session configuration is unavailable, record Unknown.

AWS documents the [IAM Identity Center idle-timeout limitation](https://docs.aws.amazon.com/pdfs/prescriptive-guidance/latest/security-reference-architecture-identity-management/security-reference-architecture-identity-management.pdf).

## Out of IaC scope

- The firewall rule review, at least every 6 months for a CIF (RTS Art. 13(h)).
- The yearly review of the network architecture and security design (RTS Art. 13(i)).
- The security measures and service levels in network service agreements (RTS Art. 13(m)).
- The procedure to assess the security of data in transit (RTS Art. 14(1)(a)).
