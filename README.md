# SOC & Network Security Home Lab

Hands-on security lab: SIEM operations, alert triage, detection engineering, network security, application security and cloud security, each documented as a report.

## Lab Setup

| Component         | Details                                                                                 |
| ----------------- | --------------------------------------------------------------------------------------- |
| SIEM              | Wazuh 4.14.8 on Ubuntu Server 24.04 (VM) · Splunk Enterprise in Docker (MacBook Air M3) |
| Detection-as-code | Sigma rules converted to Splunk SPL with sigma-cli and pySigma                          |
| Scanner           | Nmap 7.992 on macOS (MacBook Air M3)                                                    |
| Network           | Flat home network behind an ISP-provided gateway                                        |
| Cloud             | Microsoft Azure (free trial subscription). Microsoft Entra ID (cloud-only tenant)       |

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

## Detection Engineering
 
|ID|Title|Summary|
|---|---|---|
|[DE-001](0.6%20Detection%20Engineering/DE-001%20Writing%20a%20Sigma%20detection,%20converting%20it%20to%20SPL%20and%20validating%20it%20with%20TR-003.md)|Sigma detection for failed SSH logins|Wrote a Sigma rule, converted it to Splunk SPL with a custom pipeline, and validated it against the TR-003 baseline (33,253 events).|

## Application Security

| ID                                                                                                                                                                                                                              | Title                           | Summary                                                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| [AS-001](0.5%20Application%20Security/AS%20-%20001%20Dependency%20Vulnerability%20Triage%20%7C%20Fixed%20a%20critical%20Next.js%20CVE,%20reduced%2019%20%E2%86%92%2011%20vulns,%20accepted%20dev-tooling%20residual%20risk..md) | Dependency Vulnerability Triage | Fixed a critical Next.js CVE, reduced vulnerabilities from 19 to 11, and documented accepted residual risk. |
| [AS-002](0.5%20Application%20Security/AS%20-%20002%20HTTP%20security%20headers%20and%20Content%20Security%20Policy%20on%20a%20Next.js%20website.md)                                                                             | HTTP Security Headers and CSP   | Added HTTP security headers and a Content Security Policy to a Next.js website.                             |

## 0.7 Cloud Security (Microsoft Azure)
| ID     | Title                                               | Skills                                               |
| ------ | --------------------------------------------------- | ---------------------------------------------------- |
| [CS-001](https://github.com/liliaem/soc-home-lab/blob/main/0.7%20Cloud%20Security/CS-001%20%20Entra%20ID%20least%20privilege%20and%20access%20troubleshooting.md) | Entra ID least privilege and access troubleshooting | Entra ID, Azure RBAC, sign-in and audit log analysis |

## Skills Practiced
- **Security operations:** SIEM deployment (Wazuh, Splunk), alert triage, log investigation with SPL, MITRE ATT&CK mapping.
- **Detection engineering:** Sigma rules, field-mapping pipelines, validation against a known baseline, reproducible Python environments.
- **Network security:** reconnaissance and asset inventory, segmentation and default-deny firewall design.
- **Application security:** dependency vulnerability triage, HTTP security headers and CSP.
- **Reporting:** clear incident and assessment reports, sanitized before publishing.
- **Cloud security (Microsoft Azure):** Entra ID identity management, Azure RBAC and least privilege, access troubleshooting, sign-in and audit log analysis.
