# SOC Analyst Portfolio

### This repository showcases hands-on Security Operations Center (SOC) investigations, digital forensics exercises, malware analysis, and log management projects. Through realistic laboratory-based investigations, these projects demonstrate practical SOC analyst skills in log analysis, incident investigation, attacker activity analysis, memory forensics, malware analysis, indicator of compromise (IOC) development, and evidence-based reporting using industry-standard tools and methodologies.

Each folder represents a project, case or lab investigation, including the analysis process, evidence screenshots, and findings. More investigations will be added as training and practical work progress.

---

## SOC Investigations

| Case | Description | Tools / Skills |
|------|-------------|----------------|
| [VPN Connection Log Investigation](vpn-connection-log-investigation) | Investigated VPN authentication logs to identify suspicious connection patterns, failed login attempts, abnormal traffic spikes, post-termination activity, and high-volume source IPs indicative of potential unauthorized access. | Elastic Stack, KQL queries, log analysis, incident triage |
| [PowerShell-Based Phishing Intrusion & Data Exfiltration Investigation](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation) | Investigated a phishing-induced compromise involving malicious PowerShell execution, attacker-controlled activity, and data exfiltration to reconstruct the attack timeline and identify indicators of compromise (IOCs). | Linkparser, Powershell, Log analysis, Attacker activity analysis |
|[From Phishing to Persistence: Malware Investigation](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/From%20Phishing%20to%20Persistence%3A%20Malware%20Investigation) | Performed a full malware investigation of a phishing-delivered, multi-stage infection involving malicious Office macros, JavaScript payload execution, command-and-control communications, scheduled-task persistence, and memory analysis to identify attacker techniques and develop IOCs. | Volatility, Oletools, DFIR, memory forensics, malware analysis, threat hunting, process analysis, network analysis, IOC development, MITRE ATT&CK, incident response |

---
## Detection Engineering & Log Management
| Project | Description | Tools / Skills |
|---------|-------------|----------------|
| [Linux Authentication Log Monitoring with ELK](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/tree/main/Linux%20Authentication%20Log%20Monitoring%20Pipeline%20with%20Logstash%2C%20Elasticsearch%20%26%20Kibana%20(ELK)) | Built an end-to-end log ingestion pipeline that collects Linux authentication logs using Logstash, parses unstructured events with Grok, indexes structured data into Elasticsearch, and enables security investigations and threat detection through Kibana dashboards and KQL queries. | Ubuntu Linux, Logstash, Elasticsearch, Kibana, Grok, ELK Stack, SIEM, Log Analysis |

--- 
## Core Skills Demonstrated
* Security Incident Investigation
* Alert Triage
* Threat Hunting
* Log Analysis
* Malware Analysis
* Memory Forensics
* Network Traffic Analysis
* IOC Development
* MITRE ATT&CK Mapping
* Digital Forensics (DFIR)
* SIEM Investigation
* Windows & Linux Security

> ⚠️ **Note:**
## This portfolio is continuously expanding with new SOC investigations, malware analyses, DFIR exercises, detection engineering projects, and threat hunting scenarios as I continue developing practical blue-team skills.
