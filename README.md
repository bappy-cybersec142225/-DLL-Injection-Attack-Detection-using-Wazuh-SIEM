<imghttps://chatgpt.com/backend-api/estuary/content?id=file_00000000e04071fa8502ccc99cf4912c&ts=495018&p=fs&cid=1&sig=0a48eeeea1de22c704b6749931cf2553bbd28db5bbabe2aa786e5dea4f568c3e&v=0/>


# 🛡️ DLL Injection Detection Lab with Wazuh SIEM
## Threat Monitoring, Detection Engineering, and Incident Investigation for MITRE ATT&CK T1055

Your project is well structured, but for GitHub portfolio presentation and SOC/Blue Team documentation, it can be made more professional by adding objectives, attack flow, detection logic, MITRE mapping, screenshots section, and learning outcomes.

# 🛡️ DLL Injection Attack Detection using Wazuh

A Security Operations Center (SOC) lab project demonstrating the detection of DLL Injection attacks using Wazuh SIEM, Sysmon, and the MITRE ATT&CK Framework (T1055.001).

---

## 📌 Project Overview

DLL Injection is a widely used process injection technique that allows an attacker to execute malicious code within the memory space of another running process. This technique is frequently leveraged for:

* Defense Evasion
* Privilege Escalation
* Malware Execution
* Credential Theft
* Persistence

This project demonstrates how a SOC analyst can detect DLL Injection activity using:

* **Wazuh SIEM**
* **Microsoft Sysmon**
* **Custom Detection Rules**
* **MITRE ATT&CK Mapping**

A controlled attack was simulated using **InjectProc.exe**, while Sysmon generated telemetry and Wazuh correlated the events to produce security alerts.

---

# 🎯 Objectives

* Deploy a Windows endpoint monitored by Wazuh.
* Configure Sysmon for advanced process monitoring.
* Simulate DLL Injection activity.
* Collect Sysmon Event ID 8 (CreateRemoteThread).
* Generate Wazuh alerts.
* Map detections to MITRE ATT&CK T1055.001.
* Analyze attack telemetry from a SOC perspective.

---

# 🏗️ Lab Environment

| Component        | Version / Description                 |
| ---------------- | ------------------------------------- |
| Wazuh Manager    | 4.5.4                                 |
| Wazuh Agent      | 4.5.4                                 |
| Operating System | Windows 10                            |
| Sysmon           | Sysinternals Sysmon                   |
| Attack Tool      | InjectProc                            |
| Payload          | hello-world-x64.dll                   |
| Detection Source | Sysmon Event Logs                     |
| MITRE Technique  | T1055.001                             |
| MITRE Tactics    | Defense Evasion, Privilege Escalation |

---

# 🔬 MITRE ATT&CK Mapping

| Technique | Name                           |
| --------- | ------------------------------ |
| T1055     | Process Injection              |
| T1055.001 | Dynamic-link Library Injection |

### Associated Tactics

* Defense Evasion (TA0005)
* Privilege Escalation (TA0004)

---

# 🏛️ Architecture

```text
┌─────────────────────┐
│ Windows Endpoint    │
│ InjectProc.exe      │
│ hello-world-x64.dll │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Sysmon              │
│ Event ID 8          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Wazuh Agent         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Wazuh Manager       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Wazuh Dashboard     │
│ Security Alert      │
└─────────────────────┘
```

---

# 📋 Prerequisites

Install and configure the following:

* Microsoft Visual C++ Redistributable
* Sysmon
* Wazuh Agent
* Wazuh Manager
* InjectProc
* hello-world-x64.dll

---

# ⚙️ Step 1 – Install Sysmon

Install Sysmon with a monitoring configuration.

```powershell
.\sysmon64.exe -accepteula -i sysmonconfig.xml
```

Verify installation:

```powershell
Get-Service sysmon64
```

Expected Result:

```text
Status   Name
------   ----
Running  sysmon64
```

---

# ⚙️ Step 2 – Configure Wazuh Agent

Add Sysmon log collection to:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Configuration:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart the agent:

```powershell
Restart-Service Wazuh
```

---

# ⚙️ Step 3 – Configure Wazuh Detection Rules

Edit:

```bash
/var/ossec/etc/rules/local_rules.xml
```

Add:

```xml
<group name="windows,sysmon">

  <rule id="100200" level="12">
    <if_sid>61610</if_sid>
    <description>
      Possible process injection activity detected from "$(win.eventdata.sourceImage)" on "$(win.eventdata.targetImage)"
    </description>

    <mitre>
      <id>T1055.001</id>
    </mitre>
  </rule>

  <rule id="100100" level="0">
    <if_sid>100200</if_sid>
    <field name="win.eventdata.sourceImage" type="pcre2">
      (C:\\\\Windows\\\\system32)|chrome.exe
    </field>
    <description>Ignore Windows binaries and Chrome</description>
  </rule>

</group>
```

Restart Wazuh:

```bash
sudo systemctl restart wazuh-manager
```

---

# 🚨 Attack Simulation

The DLL Injection activity was simulated using InjectProc.

Command executed:

```cmd
InjectProc.exe dll_inj hello-world-x64.dll cmd.exe
```

Attack Flow:

1. InjectProc starts.
2. Target process selected (cmd.exe).
3. DLL loaded into target process.
4. Remote thread created.
5. Sysmon logs Event ID 8.
6. Wazuh receives telemetry.
7. Alert generated.

---

# 📊 Event Analysis

## Sysmon Detection

### Event ID

```text
Event ID 8
```

### Event Name

```text
CreateRemoteThread
```

### Indicators Observed

| Field          | Value          |
| -------------- | -------------- |
| Source Process | InjectProc.exe |
| Target Process | cmd.exe        |
| Start Function | LoadLibraryW   |
| Behavior       | DLL Injection  |

---

# 🛡️ Wazuh Alert Analysis

## Detection Details

| Field          | Value             |
| -------------- | ----------------- |
| Rule ID        | 100200            |
| Severity Level | 12                |
| Technique      | T1055.001         |
| Detection Type | DLL Injection     |
| Source         | Sysmon Event ID 8 |

### Generated Alert

```text
Possible process injection activity detected
```

---

# 📸 Screenshots

Include screenshots in the following order:

1. Visual C++ Installation
2. InjectProc Download
3. Payload Download
4. Wazuh Agent Deployment
5. Agent Connected to Manager
6. Sysmon Installation
7. Sysmon Verification
8. Wazuh Agent Configuration
9. Wazuh Rule Creation
10. Attack Execution
11. Sysmon Event Viewer Logs
12. Wazuh Alert Dashboard

---

# 🔍 Detection Logic

The detection relies on:

* Sysmon Event ID 8 (CreateRemoteThread)
* Source process analysis
* Target process monitoring
* Wazuh correlation rules
* MITRE ATT&CK mapping

Detection Chain:

```text
DLL Injection
     ↓
CreateRemoteThread
     ↓
Sysmon Event ID 8
     ↓
Wazuh Rule Match
     ↓
SOC Alert
```

---

# 📈 Key Findings

* DLL Injection generated Sysmon Event ID 8.
* Wazuh successfully ingested Sysmon logs.
* Custom rules detected process injection behavior.
* MITRE ATT&CK mapping provided threat context.
* SOC analysts gained visibility into process injection activity.

---

# 🎓 Lessons Learned

* Sysmon significantly improves endpoint visibility.
* Process injection techniques leave detectable artifacts.
* Wazuh can effectively correlate Sysmon telemetry.
* MITRE ATT&CK mapping enhances investigation workflows.
* Custom detection rules improve threat coverage.

---

# ✅ Conclusion

This project successfully demonstrated end-to-end detection of a DLL Injection attack using Sysmon and Wazuh. The attack simulation generated telemetry that was collected, analyzed, and mapped to MITRE ATT&CK T1055.001, providing a practical example of how SOC analysts can identify process injection techniques in real-world environments.

---

# 📚 References

* Wazuh Documentation
* Wazuh Blog – Detecting Process Injection Attacks
* Microsoft Sysinternals Sysmon
* MITRE ATT&CK Framework
* Windows Event Logging Documentation

---

## 👨‍💻 Author

**Bappy Sharma**
* IT Cybersecurity Enthusiast
