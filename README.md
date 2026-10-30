# SOC & Network Security Home Lab

Hands-on lab to practice security operations and network security: log collection, alert triage, network reconnaissance, risk assessment, and clear technical documentation.

## Lab Setup

| Component    | Details                                          |
| ------------ | ------------------------------------------------ |
| SIEM         | Wazuh v4.14.8                                    |
| Wazuh server | Ubuntu Server 24.04.5 LTS (VM)                   |
| Scanner      | Nmap 7.992 on macOS (MacBook Air M3)             |
| Network      | Flat home network behind an ISP-provided gateway |
| Log search   | Splunk Cloud (free trial), SPL                   |

## Triage Reports

| ID                                                                                                                                                   | Title                                  | Summary                                                                                                                                                                                      |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [TR-001](0.3%20Triage%20reports/TR-001%20SSH%20brute%20force%20invalid%20user.md)                                                                    | SSH Brute Force                        | Detected repeated failed SSH logins, analyzed source and pattern, documented findings and recommended actions.                                                                               |
| [TR-002](https://github.com/liliaem/soc-home-lab/blob/main/0.3%20Triage%20reports/TR-002%20Failed%20SSH%20Logins%20-%20Splunk%20vs%20Wazuh.md)       | Failed SSH Logins: Splunk vs Wazuh     | Investigated the same SSH brute-force logs in Splunk Cloud with SPL and compared detection and investigation workflow against Wazuh.                                                         |
| [TR-003](0.3%20Triage%20reports/TR-003%20SPL%20queries%20to%20investigate%20an%20SSH%20brute%20force%20attack%20in%20Splunk's%20generated%20data.md) | SSH Brute Force Investigation with SPL | Investigated 33,253 failed SSH logins from 185 IPs across 4 servers with SPL; confirmed no external IP gained access, cleared internal failures as benign, and mapped to MITRE ATT&CK T1110. |

## Network Assessments

|ID|Title|Summary|
|---|---|---|
|[NA-001](0.4%20Network%20assessments/NA-001%20Household%20Network%20Segmentation.md)|Network Segmentation|Assessed a flat home network, designed a VLAN segmentation plan with default-deny firewall policies, and evaluated options under real constraints.|
|[NA-002](0.4%20Network%20assessments/NA-002%20Network%20recon%20with%20Nmap.md)|Network Recon with Nmap|Built an asset inventory, scanned for open ports and services, and prioritized risks by severity and owner.|

## Skills Practiced

**Security operations**

- Deploying and configuring a SIEM
- Reviewing and triaging security alerts
- Log analysis and pattern recognition
- Log investigation with SPL (Splunk)

**Network security**

- Network reconnaissance and asset inventory
- Port and service analysis (attack surface)
- Network segmentation design (VLANs, zones)
- Firewall policy design (default deny)

**Reporting**

- Risk prioritization by severity and asset owner
- Writing clear incident and assessment reports
- Sanitizing reports before publishing (IPs, MACs, identifiers)