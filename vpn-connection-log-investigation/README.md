# VPN Connections Log Investigation

---

## Context

VPN connection logs were analyzed to identify suspicious user behavior and potential unauthorized access attempts.

The investigation focused on user activity, abnormal connection volumes, connection spikes, geographic login patterns, and activity associated with terminated users.

---

## Log Source

Connection logs were analyzed using the **Elastic Stack**.

The `vpn_connections` index was queried to review connection activity between **31 December 2021 and 2 February 2022**.  

Relevant fields analyzed included:

- Username  
- Source IP  
- Source country  
- Connection status  
- Timestamp  

A total of **2,861 VPN connection events** were returned during this period.

Evidence: ![Connections Timeline](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/Timeline%20log%20analysis%20(elastic).png)

---

## Investigation

### High-Volume VPN Connection Sources

Analysis of connection logs identified **238.163.231.224** as the IP responsible for the highest number of VPN connections.

Evidence: https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/high-volume-vpn-ip.png

High connection frequency from a single IP may indicate:

- Automated login attempts  
- VPN gateway testing  
- Potential brute force or misuse  

Further filtering revealed **48 connections from this IP outside the New York region**, suggesting geographically distributed access attempts.

Evidence:
![Connections Outside NY](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/connections%20outside%20NY.png)

---

### User Generating the Most VPN Traffic

Log aggregation showed that **user "James" generated the highest volume of VPN connection events**.

Evidence:![](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/top-user-traffic..png)

High connection volume from a single account can indicate:

- Legitimate heavy usage  
- Credential or account misuse  
- Automated connection scripts  

Such accounts warrant closer monitoring in a SOC environment.

---

### Targeted User Activity Analysis

Filtering VPN connection events for **user "Emanda"** revealed that **IP 107.14.1.247** generated the highest number of connection attempts for the account.

Evidence: 

![User Specific Source IP](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/emanda-source-ip.png)

This correlation helps identify:

- Regular user access patterns  
- Potential account compromise if access is unusual or unexpected  

---

### Connection Activity Spike Analysis

Time-series visualization revealed a noticeable connection spike on **January 11**.

Further log analysis identified **IP 172.201.60.191** as the primary source responsible for the surge.

Evidence:

![Connection Spike](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/jan11-traffic-spike.png)

Sudden spikes in connection activity are commonly associated with:

- Automated login or script activity  
- Credential misuse  
- Misconfigured services repeatedly attempting connection

---

### Repeated Connection Failures

Connection failure logs were analyzed to identify users experiencing repeated failed attempts.

User **Simon** was associated with the highest number of failed connection events.

Evidence:

![Failed Connections](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/failed-Connection.png)

Repeated failures may indicate:

- Brute force attempts  
- Misuse of credentials  
- User errors or misconfigurations

---

### Connection Failures in January

Filtering connection logs for **January 2022** revealed **274 failed VPN connection attempts**.

Evidence:

![January Connection Failures](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/January-Connection-failures..png)
Monitoring connection failures is critical for detecting:

- Misuse of credentials  
- Potential unauthorized access  
- Configuration issues

---

### Terminated Employee Activity

User **Johny Brown** was terminated on **January 1, 2022**.

Logs were reviewed to determine whether connection activity occurred after account deactivation.

Evidence: ![Post-Termination VPN Activity](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/vpn-connection-log-investigation/evidence/terminated-user-vpn..png)


Continued connections from terminated accounts may indicate:

- Failure to properly deactivate accounts  
- Potential credential misuse  
- Insider threat activity

---
### Query-Based Analysis: Geographic and User Filtering

A query was executed to identify connection events originating from the **United States** involving specific users: **James** or **Albert**.

Evidence:![](https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/%22vpn-connection-log-investigation/evidence/Source_Country%20%3A%20%5C%22United%20States%5C%22%20and%20UserName%20%3A%20%5C%22Albert%5C%22%20or%20UserName%20%3A%20%5C%22James%5C%22%20.png%22)




## Findings

The investigation identified several connection patterns requiring monitoring:

- High-volume connections from **238.163.231.224**  
- Connection spike on **January 11** linked to **172.201.60.191**  
- Elevated failed connection attempts associated with user **Simon**  
- **274 failed connection attempts** in January  
- Continued activity from a **terminated employee account**

These patterns demonstrate how **connection log analysis** can help detect potential account misuse, automated login attempts, and unauthorized access.

---

## Skills Demonstrated

- VPN connection log analysis  
- Security event triage  
- SIEM query construction  
- User and IP anomaly investigation  
- Security log pattern analysis
