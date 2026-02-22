#  Azure Sentinel & Cowrie Honeypot: Attack Kill Chain Visualization

This project demonstrates a **cloud‑native security monitoring pipeline** using a Cowrie SSH Honeypot hosted on Azure and fully integrated with **Microsoft Sentinel**.  
The objective of this project is to move beyond basic log ingestion and build a **complete Attack Kill Chain visualization**, enabling real‑time tracking of adversary behavior.

---

##  Project Overview

Instead of creating custom tables, this project leverages the **standard Syslog stream** collected via the **Azure Monitor Agent (AMA)**.  
By using KQL to parse raw Syslog strings, I transformed noisy low‑level system logs into **high‑fidelity attacker telemetry**.

### **Key Components**
- **Honeypot:** Cowrie (SSH/Telnet) listening on port `2222`  
- **Ingestion:** Azure Monitor Agent (AMA) collecting Syslogs  
- **SIEM:** Microsoft Sentinel  
- **Detection Logic:** 6 custom analytical rules mapped to **MITRE ATT&CK**

---

##  Setup & Configuration

### **1. Cowrie Configuration**

To enable logging to Syslog, the following section was added to `cowrie.cfg`:

```ini
[output_localsyslog]
enabled = true
facility = USER
format = json
```

### 2. Port Redirection (22 → 2222)
Since Cowrie listens on port 2222 by default to avoid running as root, I utilized `iptables` to silently redirect all traffic from the standard SSH port (22) to the honeypot port. This ensures attackers believe they are hitting a standard SSH service.

```bash
# Redirect incoming TCP traffic on port 22 to port 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```
### 3. Data Parsing (The KQL Engine)

To transform raw, semi-structured Syslog data into a queryable format, I developed a custom KQL Parser. This logic extracts nested JSON fields from the `SyslogMessage` column, allowing for granular analysis of attacker behavior.
```kusto
Syslog
| where ProcessName == "cowrie"
| extend LogData = parse_json(SyslogMessage)
| project 
    TimeGenerated, 
    SourceIP = tostring(LogData.src_ip),
    User = tostring(LogData.username),
    Pass = tostring(LogData.password),
    Command = tostring(LogData.input),
    EventID = tostring(LogData.eventid)

```
### 4. Honeypot Attack Kill Chain
I designed and deployed **6 Analytical Rules** in Microsoft Sentinel. Each rule is mapped to specific **MITRE ATT&CK** techniques to categorize activity into a logical attack progression:

* **Reconnaissance:** `SSH Brute Force Threshold Exceeded` (**T1110**) – Identifying automated scanning and password spraying.
* **Initial Access:** `Successful Attacker Login` (**T1078**) – Alerting on successful credential guesses.
* **Execution:** `Suspicious Command Executed` (**T1059**) – Capturing fingerprinting and post-exploitation commands.
* **Persistence:** `Hardened Persistence` (**T1136**) – Detecting attempts to create backdoors or new users.
* **Defense Evasion:** `Defense Evasion Activity` (**T1070**) – Monitoring for log-clearing or history deletion.
* **Impact:** `Malware Payload Uploaded` (**T1105**) – Flagging the download and execution of malicious binaries.



---

### 5. Visualization
The **Honeypot Attack Progression** workbook provides a real-time, high-level overview of the threat landscape. It visualizes the "funnel" of activity, showing how many unique threats progress from initial "Reconnaissance" through to the final "Impact" stage.



---

### 6. Key Takeaways
* **Syslog vs. Custom Tables:** Utilizing the standard **Syslog** table simplifies the ingestion pipeline and maintains a cloud-native architecture without the overhead of managing custom schema deployments.
* **Active Defense:** A honeypot acts as a high-fidelity sensor. Because there is no legitimate reason for traffic to hit these ports, any alert generated is 100% actionable, eliminating the "alert fatigue" common in production environments.
* **Visualizing the Story:** Mapping raw alerts to a structured Kill Chain provides immediate context for incident response and a deeper strategic understanding of adversary TTPs (Tactics, Techniques, and Procedures).





