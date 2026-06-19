
# 🛡️ DLL Injection Attack Detection using Wazuh SIEM
https://github.com/bappy-cybersec142225/-DLL-Injection-Attack-Detection-using-Wazuh-SIEM/blob/main/SOC%20LAB.png
![Uploading SOC LAB.png…]()

## 📌 Project Overview

This project demonstrates how to detect **DLL Injection attacks** in a Windows environment using **Wazuh SIEM**, **Sysmon**, and log analysis techniques. It simulates a real-world SOC detection scenario where malicious code injection into legitimate processes is identified and alerted in real time.

The lab includes:

* Wazuh Manager (SIEM Server)
* Windows 11 Victim Machine with Sysmon
* Attack Simulation using Process Injection techniques
* Detection Rules & Alerting in Wazuh Dashboard

---

## 🎯 Objectives

* Understand DLL Injection attack behavior
* Configure Sysmon for detailed Windows event logging
* Forward logs to Wazuh SIEM
* Create detection rules for process injection activity
* Visualize alerts in Wazuh Dashboard

---

## 🧱 Lab Architecture

```
Attacker Machine (Atomic Red Team)
            ↓
Windows 11 (Sysmon Installed - Victim)
            ↓
Wazuh Agent
            ↓
Wazuh Manager + Dashboard (Ubuntu Server)
```

---

## 🖥️ Technologies Used

* Wazuh SIEM (Latest Version)
* Sysmon (Sysinternals)
* Windows 11 VM
* Ubuntu Server (Wazuh Manager)
* Atomic Red Team (Attack Simulation)
* VirtualBox / VMware

---

## ⚙️ Prerequisites

* VirtualBox / VMware installed
* 8GB+ RAM recommended
* Ubuntu Server (Wazuh)
* Windows 11 VM
* Internet connection for package installation

---

## 📥 Step 1: Install Wazuh Server

Install Wazuh using the official installer:

```bash
https://github.com/samlul008ghub/soc_setup/
sudo systemctl start elasticsearch
```

After installation access dashboard:

```
https://<WAZUH_SERVER_IP:5601>
```

---

## 🪟 Step 2: Install Sysmon on Windows 11

Download Sysmon:

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

Install with config:

```powershell
sysmon64.exe -accepteula -i sysmonconfig.xml
```

Recommended config:

* SwiftOnSecurity Sysmon config

---

## 📡 Step 3: Install Wazuh Agent on Windows

Download agent:

```
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
```

Configure manager IP in:

```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Start service:

```powershell
net start wazuh
```

---

## 🧪 Step 4: DLL Injection Attack Simulation

Using Atomic Red Team:

```powershell
Invoke-AtomicTest T1055
```

Or manual simulation:

* Use process injection techniques
* Inject DLL into explorer.exe or notepad.exe

---

## 🔍 Step 5: Wazuh Detection Rule (DLL Injection)

Add custom rule in Wazuh Manager:

📍 File location:

```
/var/ossec/etc/rules/local_rules.xml
```

### 🧾 Detection Rule Example

```xml
<group name="windows,sysmon,process_injection,">
  <rule id="100101" level="12">
    <if_sid>61603</if_sid>
    <field name="win.eventdata.injectiontype">CreateRemoteThread</field>
    <description>Possible DLL Injection detected (CreateRemoteThread method)</description>
    <mitre>
      <id>T1055</id>
    </mitre>
  </rule>
</group>
```

---

## 🔄 Restart Wazuh Manager

```bash
systemctl restart wazuh-manager
```

---

## 📊 Step 6: Detection in Dashboard

Go to:

```
Wazuh Dashboard → Security Events
```

Filter:

```
rule.description: DLL Injection
```

---

## 📸 Screenshots Section (IMPORTANT)

> Add screenshots in your GitHub repo under `/Screenshots` folder

### 🔹 Wazuh Dashboard and Server

![Wazuh server](https://github.com/bappy-cybersec142225/-DLL-Injection-Attack-Detection-using-Wazuh-SIEM/tree/main/Screenshots/Wazuah%20Server)

### 🔹 Sysmon Logs

![Sysmon Logs](https://github.com/bappy-cybersec142225/Splunk-SOC-Lab/tree/main/screenshots/Sysmon)

### 🔹 Attack Execution (Atomic Red Team)

![Attack Simulation](https://github.com/bappy-cybersec142225/Splunk-SOC-Lab/tree/main/screenshots/Atomic-Red-Team)

### 🔹 Custom Rule in Wazuh

![Custom Rule](https://github.com/bappy-cybersec142225/-DLL-Injection-Attack-Detection-using-Wazuh-SIEM/tree/main/Screenshots/Wazuh%20Agent)

---

## 📈 Results

* DLL Injection activity successfully detected
* Sysmon logs properly forwarded to Wazuh
* Custom rule triggered high severity alert
* SOC-style detection workflow validated

---

## 🧠 MITRE ATT&CK Mapping

| Technique         | ID    | Description                       |
| ----------------- | ----- | --------------------------------- |
| Process Injection | T1055 | DLL Injection into remote process |

---

## 🚀 Future Improvements

* Add behavioral analytics (UEBA)
* Integrate Sigma rules
* Add ELK correlation dashboards
* Automate attack simulation pipeline

---

## 👨‍💻 Author

**Bappy Sharma**
IT Audit & SOC Enthusiast

---

## 📌 Disclaimer

This project is for **educational and SOC training purposes only**.

