# SOC & Network Security Home Lab

Hands-on security lab: SIEM operations, alert triage, detection engineering, network security and application security, each documented as a report.

## Lab Setup

| Component         | Details                                                                                 |
| ----------------- | --------------------------------------------------------------------------------------- |
| SIEM              | Wazuh 4.14.8 on Ubuntu Server 24.04 (VM) · Splunk Enterprise in Docker (MacBook Air M3) |
| Detection-as-code | Sigma rules converted to Splunk SPL with sigma-cli and pySigma                          |
| Scanner           | Nmap 7.992 on macOS (MacBook Air M3)                                                    |
| Network           | Flat home network behind an ISP-provided gateway                                        |

## Triage Reports

| ID                                                                                                                                                   | Title                                  | Summary                                                                                                                                                                                      |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [TR-001](0.3%20Triage%20reports/TR-001%20SSH%20brute%20force%20invalid%20user.md)                                                                    | SSH Brute Force                        | Detected repeated failed SSH logins, analyzed source and pattern, documented findings and recommended actions.                                                                               |
| [TR-002](https://github.com/liliaem/soc-home-lab/blob/main/0.3%20Triage%20reports/TR-002%20Failed%20SSH%20Logins%20-%20Splunk%20vs%20Wazuh.md)       | Failed SSH Logins: Splunk vs Wazuh     | Investigated the same SSH brute-force logs in Splunk Cloud with SPL and compared detection and investigation workflow against Wazuh.                                                         |
| [TR-003](0.3%20Triage%20reports/TR-003%20SPL%20queries%20to%20investigate%20an%20SSH%20brute%20force%20attack%20in%20Splunk's%20generated%20data.md) | SSH Brute Force Investigation with SPL | Investigated 33,253 failed SSH logins from 185 IPs across 4 servers with SPL; confirmed no external IP gained access, cleared internal failures as benign, and mapped to MITRE ATT&CK T1110. |

## Network Assessments

| ID                                                                                   | Title                   | Summary                                                                                                                                            |
| ------------------------------------------------------------------------------------ | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| [NA-001](0.4%20Network%20assessments/NA-001%20Household%20Network%20Segmentation.md) | Network Segmentation    | Assessed a flat home network, designed a VLAN segmentation plan with default-deny firewall policies, and evaluated options under real constraints. |
| [NA-002](0.4%20Network%20assessments/NA-002%20Network%20recon%20with%20Nmap.md)      | Network Recon with Nmap | Built an asset inventory, scanned for open ports and services, and prioritized risks by severity and owner.                                        |
## Application Security

|ID|Title|Summary|
|---|---|---|
|AS-001|Dependency Vulnerability Triage|Fixed a critical Next.js CVE, reduced vulnerabilities from 19 to 11, and documented accepted residual risk.|
|AS-002|HTTP Security Headers and CSP|Added HTTP security headers and a Content Security Policy to a Next.js website.|
## Skills Practiced
- **Security operations:** SIEM deployment (Wazuh, Splunk), alert triage, log investigation with SPL, MITRE ATT&CK mapping.
- **Detection engineering:** Sigma rules, field-mapping pipelines, validation against a known baseline, reproducible Python environments.
- **Network security:** reconnaissance and asset inventory, segmentation and default-deny firewall design.
- **Application security:** dependency vulnerability triage, HTTP security headers and CSP.
- **Reporting:** clear incident and assessment reports, sanitized before publishing.