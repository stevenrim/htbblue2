# 🔥 HTB CampFire-2 — Forensic Investigation of AS-REP Roasting

## 🧨 The Problem  
Forela's security systems detected an old domain admin account attempting to request a Kerberos ticket from the Domain Controller (DC). This account was flagged as inactive in inventory, prompting suspicion of an **AS-REP Roasting attack** — where an attacker targets user accounts with Kerberos pre-authentication disabled. The objective of this analysis was to confirm the presence of such an attack, identify the targeted account, and trace attacker activity within the provided logs.

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| **Event Viewer** | Manual log inspection and event correlation |
| **EvtxECmd (Eric Zimmerman)** | CLI-based EVTX parsing (alternative method) |
| **Splunk / ELK (Suggested)** | Scalable SIEM analysis for large log sets |
| **Windows Security Logs** | Core artifact source (Event IDs 4768 & 4769) |

## 🔍 Method (Investigation Steps)

1. **Initial Log Review**  
   - Verified the provided `.zip` and extracted a Windows Security log from the Domain Controller.  
   - Opened the EVTX file using **Event Viewer** for targeted inspection.  
   - Confirmed the hash of the `.zip` for integrity before proceeding with analysis.

2. **Detection of AS-REP Roasting Activity**  
   - Filtered for **Event ID 4768**, which logs Kerberos authentication ticket requests.  
   - Focused on events with these specific indicators:  
     - `Pre-authentication type = 0` (indicates it's disabled)  
     - `Ticket encryption type = 0x17 (RC4)`  
     - `Service Name = krbtgt`  
   - Identified the log entry where these conditions were all present, signaling an AS-REP Roast attempt.

3. **Identifying the Targeted User**  
   - From the same event, extracted the `Account Name` (victim user) and `SID`.  
   - Noted the `Client Address` field to determine the **internal IP address** of the attacker’s machine.

4. **Tracing the Source User of the Attack**  
   - Investigated subsequent log entries, particularly **Event ID 4769**, to find additional context.  
   - Cross-referenced the previously identified IP address with the next event.  
   - Discovered another user account active on the same IP, strongly suggesting it was used to conduct the attack.  
   - This helped differentiate between the **victim user** (targeted for AS-REP Roast) and the **attacker’s compromised user account**.

## ✅ The Solution  
By examining Windows Security logs, we confirmed that an AS-REP Roasting attack had occurred. A domain account with pre-authentication disabled was targeted, and its ticket request was captured using RC4 encryption — suitable for offline cracking. Further analysis revealed which account initiated the attack, along with its originating IP address, supporting a clear attribution chain. This lab emphasizes the value of Event ID correlation and log timeline reconstruction in credential-based attack detection.

> 🧠 *Even in small datasets, clear log chaining and attention to Kerberos-specific indicators can reveal significant attacker behavior. This lab illustrates a common but subtle AD attack vector and demonstrates how to identify it using native Windows tools.*

## 🧭 Framework Mapping

🔶 **MITRE ATT&CK**

| Tactic               | Technique                          | ID         | Description                                           |
|----------------------|-------------------------------------|------------|-------------------------------------------------------|
| **Credential Access**| AS-REP Roasting                    | T1558.004  | Abuse of Kerberos ticket requests without pre-auth    |
| **Discovery**        | Account Discovery (AD Enumeration) | T1087.002  | Identifying accounts with weak Kerberos configurations|
| **Defense Evasion**  | Valid Account Abuse                | T1078      | Reuse of a compromised domain account                 |

🟦 **NIST Cybersecurity Framework (CSF)**

| Function   | Category             | Subcategory                                     |
|------------|----------------------|-------------------------------------------------|
| **Detect** | Anomalies and Events | Identify abnormal Kerberos activity             |
| **Respond**| Analysis             | Analyze the event to determine scope and impact |
| **Respond**| Mitigation           | Contain compromised accounts and affected hosts |

🌐 **ISO/IEC 27001:2013**

| Control Area            | Control ID | Description                                  |
|--------------------------|------------|----------------------------------------------|
| **Access Control**       | A.9.2      | Enforcing authentication mechanisms          |
| **Operations Security**  | A.12.4     | Event logging and monitoring                 |
| **Incident Management**  | A.16.1     | Incident analysis and containment            |


## ⚠ Disclaimer
```
All activities, tools, and techniques presented in this portfolio were performed exclusively within legally authorized,
controlled environments for the purpose of cybersecurity education and ethical research. This work adheres to established
ethical hacking standards, relevant legal regulations, and responsible disclosure protocols. Any application of these
methods outside of approved environments without explicit authorization is strictly prohibited and may be unlawful.

This repository aligns with HackTheBox’s content usage policies. This write-up is based on a retired Hack The Box
room called CampFire-1. It does not contain walkthroughs, solutions, or flag disclosures. All content is intended to
demonstrate professional development and applied learning through high-level summaries and overviews.
```


