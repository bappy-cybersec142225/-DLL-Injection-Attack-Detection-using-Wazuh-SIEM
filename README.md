<img width="1536" height="1024" alt="ChatGPT Image Jun 22, 2026, 12_05_13 AM" src="https://github.com/user-attachments/assets/094ece67-8a3f-45ce-a691-48887f6c2dd0" />!Uploading ChatGPT Image Jun 22, 2026, 12_05_13 AM.png…

# 🛡️ DLL Injection Attack Detection Using Wazuh

A cybersecurity lab project demonstrating DLL Injection attack detection using **Wazuh**, **Sysmon**, and **MITRE ATT&CK T1055.001** techniques.

---

## Introduction

Dynamic-Link Library (DLL) injection is a process injection technique used by attackers to execute malicious code within the address space of another process. This technique is commonly associated with **Privilege Escalation** and **Defense Evasion** activities and is categorized under **MITRE ATT&CK T1055.001 – Dynamic-link Library Injection**.

This project demonstrates how **Wazuh** and **Sysmon** can be used to detect DLL injection activity in a Windows environment. A controlled lab environment was created to simulate the attack using **InjectProc**, while Sysmon generated the corresponding events and Wazuh correlated and generated alerts based on the observed behavior.

---

## Lab Environment

| Component        | Description            |
| ---------------- | ---------------------- |
| Wazuh Server     | Version 4.5.4          |
| Wazuh Agent      | Windows Agent 4.5.4    |
| Operating System | Windows 10             |
| Sysmon           | Sysinternals Sysmon    |
| Attack Tool      | InjectProc             |
| Payload          | hello-world-x64.dll    |
| Technique        | MITRE ATT&CK T1055.001 |

---

## Architecture

```text
Windows Endpoint
       │
       ▼
InjectProc.exe + hello-world-x64.dll
       │
       ▼
Sysmon Event Logs
       │
       ▼
Wazuh Agent
       │
       ▼
Wazuh Manager
       │
       ▼
Wazuh Dashboard Alert
```

---

## Prerequisites

Before performing the attack simulation, install and configure the following components:

* Microsoft Visual C++ Redistributable
* InjectProc
* hello-world-x64.dll
* Sysmon
* Wazuh Agent
* Wazuh Manager 4.5.4

---

# Installation & Configuration

This section describes the step-by-step setup of the lab environment used to simulate and detect DLL Injection attacks.

---

## Step 1: Install Microsoft Visual C++ Redistributable

Install the required Visual C++ runtime on the Windows endpoint to ensure proper execution of InjectProc and related binaries.

---

## Step 2: Download InjectProc Tool

Download the **InjectProc** tool and the DLL payload (**hello-world-x64.dll**) and place them in a dedicated working directory.

### Directory Structure

```text
C:\Users\fazleh\Downloads\DLLInjection\
│
├── InjectProc.exe
├── hello-world-x64.dll
```

---

## Step 3: Install and Configure Wazuh Agent

Install the Wazuh agent on the Windows endpoint and register it with the Wazuh Manager.

Verify the agent is successfully connected to the manager before continuing.

---

## Step 4: Install Sysmon

Install Sysmon using Sysinternals Sysmon and a predefined configuration file.

### Installation Command

```powershell
.\sysmon64.exe -accepteula -i sysmonconfig.xml
```

### Verify Installation

```powershell
Get-Service sysmon64
```

Expected output should show the Sysmon service in the **Running** state.

---

## Step 5: Integrate Sysmon Logs with Wazuh

Edit the Wazuh Agent configuration file (`ossec.conf`) and add the following configuration:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart the Wazuh agent to apply the changes.

---

## Step 6: Create Custom Detection Rules in Wazuh

Edit the local rules file on the Wazuh Manager:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

Add the following rules:

```xml
<group name="windows,sysmon">

  <rule id="100200" level="12">
    <if_sid>61610</if_sid>
    <description>
      Possible process injection activity detected from "$(win.eventdata.sourceImage)"
      on "$(win.eventdata.targetImage)"
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
    <description>
      Ignore Windows binaries and Chrome
    </description>
  </rule>

</group>
```

Restart the Wazuh Manager after saving the configuration.

```bash
sudo systemctl restart wazuh-manager
```

---

# Attack Simulation

After completing the setup, execute the DLL injection attack using InjectProc.

### Attack Command

```cmd
InjectProc.exe dll_inj hello-world-x64.dll cmd.exe
```

This injects the DLL into the `cmd.exe` process using the `LoadLibraryW` technique.

---

# Sysmon Event Analysis

Sysmon generates **Event ID 8 (CreateRemoteThread)**, which is a strong indicator of process injection activity.

### Key Observations

| Field          | Value                  |
| -------------- | ---------------------- |
| Source Process | InjectProc.exe         |
| Target Process | cmd.exe                |
| Technique      | LoadLibraryW Injection |
| Event ID       | 8                      |
| Event Type     | CreateRemoteThread     |

---

# Wazuh Detection

Wazuh successfully detected the DLL injection behavior and generated an alert based on the custom rule.

### Alert Details

| Property        | Value                                 |
| --------------- | ------------------------------------- |
| Rule ID         | 100200                                |
| Alert Level     | 12                                    |
| MITRE Technique | T1055.001                             |
| Technique Name  | Dynamic-link Library Injection        |
| Tactics         | Defense Evasion, Privilege Escalation |

---

# Detection Workflow

```text
DLL Injection
      │
      ▼
Sysmon Event ID 8
(CreateRemoteThread)
      │
      ▼
Wazuh Agent
      │
      ▼
Wazuh Custom Rule 100200
      │
      ▼
MITRE ATT&CK Mapping
(T1055.001)
      │
      ▼
Dashboard Alert
```

---

# Key Findings

* DLL injection activity generated Sysmon Event ID 8.
* Wazuh successfully analyzed Sysmon telemetry.
* Process injection behavior was detected and classified.
* Wazuh mapped the activity to MITRE ATT&CK T1055.001.
* Security analysts gained visibility into Defense Evasion and Privilege Escalation techniques.

---

# Conclusion

This lab demonstrated how DLL Injection attacks can be detected using **Sysmon** for telemetry generation and **Wazuh** for log analysis and alerting.

The implementation successfully mapped malicious behavior to the MITRE ATT&CK framework, providing valuable visibility into process injection techniques commonly used by attackers. This approach can help Security Operations Center (SOC) teams improve detection coverage for advanced endpoint threats.

---

# References

* Wazuh Documentation
* Microsoft Sysinternals Sysmon
* MITRE ATT&CK Framework
* Wazuh Blog – Detecting Process Injection Attacks with Wazuh

---

## MITRE ATT&CK Mapping

| Technique ID | Technique                      |
| ------------ | ------------------------------ |
| T1055.001    | Dynamic-link Library Injection |

| Tactic               |
| -------------------- |
| Defense Evasion      |
| Privilege Escalation |

---


**Focus Areas:**

* Wazuh SIEM
* Sysmon Monitoring
* Endpoint Detection
* MITRE ATT&CK Mapping
* Threat Detection Engineering

## Screenshots:

1. Sysmon Event ID 8 log
2. Wazuh Dashboard alert
3. Agent status page
4. Architecture diagram
5. MITRE ATT&CK mapping screenshot

## Author

Cybersecurity SOC Detection Engineering Lab Project

