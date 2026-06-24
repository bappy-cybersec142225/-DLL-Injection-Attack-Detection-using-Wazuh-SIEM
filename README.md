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
<img width="986" height="696" alt="Download MS Visual C++" src="https://github.com/user-attachments/assets/6a000e22-cbdd-4a7e-848c-0ed93a311f20" />

<img width="481" height="297" alt="Install MS Visual C++" src="https://github.com/user-attachments/assets/d406b8fa-286a-44d3-9cec-62c81505b8ac" />

# Installation & Configuration

This section describes the step-by-step setup of the lab environment used to simulate and detect DLL Injection attacks.

---

## Step 1: Install Microsoft Visual C++ Redistributable

Install the required Visual C++ runtime on the Windows endpoint to ensure proper execution of InjectProc and related binaries.

---

## Step 2: Download InjectProc Tool

Download the **InjectProc** tool and the DLL payload (**hello-world-x64.dll**) and place them in a dedicated working directory.
<img width="481" height="297" alt="Install MS Visual C++" src="https://github.com/user-attachments/assets/68124bdc-61fb-4e70-a77d-e059ab0dff2f" />
<img width="962" height="623" alt="Download Hellow world ddl" src="https://github.com/user-attachments/assets/4b6b3856-8afe-41ae-8dcd-1df9f2e5711d" />
<img width="657" height="190" alt="Download Hello world ddl" src="https://github.com/user-attachments/assets/f8a9f7b9-00e1-4805-9801-c9a1350ece3b" />


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
<img width="1482" height="1912" alt="Wazuh agent deploy process" src="https://github.com/user-attachments/assets/0152a9f0-d336-4348-9824-4af95a8b3d0d" />
<img width="659" height="593" alt="Wazuh agent active endpoint" src="https://github.com/user-attachments/assets/e39daab3-671d-4bb4-ac64-d0bd1353bc30" />

Verify the agent is successfully connected to the manager before continuing.

<img width="1912" height="518" alt="Wazuh agent active" src="https://github.com/user-attachments/assets/94b1219d-e5ed-4417-afb6-64a2ad20603f" />

---

## Step 4: Install Sysmon

Install Sysmon using Sysinternals Sysmon and a predefined configuration file.
<img width="620" height="382" alt="sysmon install verify" src="https://github.com/user-attachments/assets/40bc4ebc-ed86-43e9-81c9-1a73cc993765" />
<img width="922" height="831" alt="result in Eventviewer" src="https://github.com/user-attachments/assets/559703fc-72d7-4844-83c9-913796a71b2f" />

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
<img width="721" height="501" alt="wazuh agent ossec file edit for sysmon log" src="https://github.com/user-attachments/assets/5dc6277c-f5b2-4ddf-8e22-3f32b525cc1e" />
<img width="320" height="280" alt="wazuh agent restart after edit" src="https://github.com/user-attachments/assets/f340cbe9-f378-4778-a559-7de1cd41ed7a" />

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
<img width="1898" height="862" alt="wazuh deshboar event chack" src="https://github.com/user-attachments/assets/90c880f0-a51f-4a61-8228-4655af587b11" />

---

# Attack Simulation

After completing the setup, execute the DLL injection attack using InjectProc.

### Attack Command

```cmd
InjectProc.exe dll_inj hello-world-x64.dll cmd.exe
```
<img width="979" height="480" alt="image" src="https://github.com/user-attachments/assets/2c533b7a-5dc1-434c-91b2-e7a472c6fafe" />
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
<img width="971" height="483" alt="add rules in wazuh manager" src="https://github.com/user-attachments/assets/6c0a02d0-7372-4074-a299-96f7b8efa49b" />
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

