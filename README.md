# SOC Analyst Portfolio

This repository contains hands-on SOC investigations, training exercises, and practical labs demonstrating skills in **incident response, threat detection, log analysis, alert triage, malware analysis, and network & memory investigations**.

Each folder represents a case or lab investigation, including the analysis process, evidence screenshots, and findings. More investigations will be added as training and practical work progress.

---

## Investigation Cases

| Case | Description | Tools / Skills |
|------|-------------|----------------|
| [VPN Connection Log Investigation](vpn-connection-log-investigation) | Analysis of VPN connection logs to identify abnormal connection patterns, traffic spikes, failed connection attempts, post-termination activity, and high-volume source IPs. | Elastic Stack, KQL queries, log analysis, incident triage |
| [PowerShell-Based Phishing Intrusion & Data Exfiltration Investigation](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation) | Analysis of a phishing-based compromise that resulted in malicious PowerShell execution, attacker-controlled activity, and data exfiltration from a Windows workstation. | Linkparser, Powershell, Log analysis, Attacker activity analysis |
|[From Phishing to Persistence: Malware Investigation](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/From%20Phishing%20to%20Persistence%3A%20Malware%20Investigation) | Investigation of a phishing-delivered multi-stage malware infection involving malicious macros, JavaScript payload execution, command-and-control communications, and scheduled-task persistence. | Volatility, Oletools, DFIR, memory forensics, malware analysis, threat hunting, process analysis, network analysis, IOC development, MITRE ATT&CK, incident response |

---
## Log Management
| Project | Description | Tools / Skills |
|---------|-------------|----------------|
| [Linux Authentication Log Monitoring with ELK](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/Linux%20Authentication%20Log%20Monitoring%20Pipeline%20with%20Logstash%2C%20Elasticsearch%20%26%20Kibana%20(ELK)) | Built an end-to-end log ingestion pipeline using Logstash to collect Linux authentication logs, parse unstructured events with Grok, index them into Elasticsearch, and investigate authentication activity in Kibana. | Ubuntu Linux, Logstash, Elasticsearch, Kibana, Grok, ELK Stack, SIEM, Log Analysis |

> ⚠️ **Note:** Additional investigations, labs, and practical exercises will be added here as they are completed. This portfolio will grow to demonstrate a wide range of SOC skills and real-world log investigation experience.
