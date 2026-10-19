# SOC & Network Security Home Lab

Hands-on lab to practice security operations and network security: log collection, alert triage, network reconnaissance, risk assessment, and clear technical documentation.

## Lab Setup

|Component|Details|
|---|---|
|SIEM|Wazuh v4.14.8|
|Wazuh server|Ubuntu Server 24.04.5 LTS (VM)|
|Scanner|Nmap 7.992 on macOS (MacBook Air M3)|
|Network|Flat home network behind an ISP-provided gateway|

## Triage Reports

|ID|Title|Summary|
|---|---|---|
|[TR-001](https://claude.ai/chat/0.3%20Triage%20reports/TR-001%20SSH%20brute%20force%20invalid%20user.md)|SSH Brute Force|Detected repeated failed SSH logins, analyzed source and pattern, documented findings and recommended actions.|

## Network Assessments

|ID|Title|Summary|
|---|---|---|
|[NA-001](https://claude.ai/chat/0.4%20Network%20assessments/NA-001%20Household%20Network%20Segmentation.md)|Network Segmentation|Assessed a flat home network, designed a VLAN segmentation plan with default-deny firewall policies, and evaluated options under real constraints.|
|[NA-002](https://claude.ai/chat/0.4%20Network%20assessments/NA-002%20Network%20recon%20with%20Nmap.md)|Network Recon with Nmap|Built an asset inventory, scanned for open ports and services, and prioritized risks by severity and owner.|

## Skills Practiced

**Security operations**

- Deploying and configuring a SIEM
- Reviewing and triaging security alerts
- Log analysis and pattern recognition

**Network security**

- Network reconnaissance and asset inventory
- Port and service analysis (attack surface)
- Network segmentation design (VLANs, zones)
- Firewall policy design (default deny)

**Reporting**

- Risk prioritization by severity and asset owner
- Writing clear incident and assessment reports
- Sanitizing reports before publishing (IPs, MACs, identifiers)