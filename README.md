# SOC Lab Setup using Wazuh SIEM, Suricata & Windows Agent

## Project Overview
This project demonstrates a complete Security Operations Center (SOC) lab environment using open-source tools for log collection, intrusion detection, and security monitoring.

The lab integrates:
* Wazuh (SIEM & log analysis)
* Elasticsearch (log storage & indexing)
* Kibana (visual monitoring)
* Filebeat (log forwarding)
* Suricata (NIDS for traffic monitoring)

The environment simulates real-world attack detection using a Windows 10 agent and Kali Linux SIEM server.

## Project Objectives
* Deploy a complete SIEM stack for security monitoring
* Collect logs from a Windows 10 endpoint
* Detect malicious activities such as process injection and network scanning
* Integrate Suricata IDS with Wazuh
* Visualize security events using Kibana dashboards

## System Requirements
### 🖥️ Host Machine
RAM: 8GB minimum (16GB recommended)
CPU: 4 cores minimum
Storage: 50GB free
Virtualization enabled (VT-x / AMD-V)
### 🖥️ Virtual Machines
🔵 SIEM Server (Kali Linux / Ubuntu)
* 4 CPU cores
* 6–8 GB RAM
* 30–40 GB storage
🟢 Endpoint Machine (Windows 10)
* 2 CPU cores
* 4 GB RAM
* 25 GB storage

## 🌐 Network Configuration
* Use Bridged Adapter or Host-only network
* Ensure both machines can ping each other
* Windows agent must connect to SIEM server IP

## ⚙️ Installation Steps
1. Clone the Repository
```bash
git clone https://github.com/samiul008ghub/soc_setup/
cd soc_setup
chmod +x setup_script.sh
```
2. Run Setup Script
 ```bash
./setup_script.sh
```
Follow on-screen prompts to install:

* Wazuh Manager
* Elasticsearch
* Kibana
* Filebeat
* Suricata (NIDS)

## 🌐 Access Wazuh Dashboard (Kibana)

After successful installation, the Wazuh dashboard (Kibana) can be accessed through a web browser.

### 🔗 URL: https://Your_IP:5601 OR https://localhost:5601

## Login Credentials
- Use the credentials generated during installation:

* Username: elastic (or as configured in setup script)
* Password: (provided during installation or setup output)

![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/07b1e21df68642055cd24bbfabcb4a3b653e3eb4/images/8.png)

## Install Sysmon (Windows) && sysmonxml file 
Sysmon is required for advanced Windows event logging and process injection detection.

### Step 1: Download Sysmon
Download Sysmon from Microsoft Sysinternals:
[Sysinternals Sysmon](https://download.sysinternals.com/files/Sysmon.zip)

### Step 2: Download symon.xml file
Download Symon.xml file from:
[sysmonconfig.xml](https://wazuh.com/resources/blog/emulation-of-attack-techniques-and-detection-with-wazuh/sysmonconfig.xml)

## DLL Injection (Download & Preparation)
**Note:** Important Notice
This section is part of a controlled cybersecurity lab environment for educational purposes only.
All activities should be performed inside a Virtual Machine (Windows 10 lab environment).

### Step 1 — Download Required Tools

Noted: First we need to turn off real time protection of our windows machine.

For simulating DLL Injection attack detection, you need the following tool:
InjectProc Tool - This tool is used for simulating process injection attacks in a lab environment.

📌 Download Source:
Use the official reference from Wazuh documentation:
1. [ Microsoft Visual C++](https://www.microsoft.com/en-us/download/details.aspx?id=53840)
2. [InjectProc](https://github.com/secrary/InjectProc/releases)
3. [hello-world-x64.dll](hello-world-x64.dll)

Before starting the setup, all required tools and configurations must be downloaded and installed on both the Server (Kali/Ubuntu) and the Windows Wazuh Agent machine.

## Suricata + Wazuh Rule Configuration (Kali Linux)
## Objective:
This section describes how to configure Suricata custom rules and Wazuh detection rules for detecting network threats and process injection activity.

### Step 1 — Switch to Root User
```bash
sudo su
```
### Step 2 — Navigate to Suricata Configuration
```bash
cd /etc/suricata
```
### Step 3 — Configure Suricata YAML File
Open configuration file:
```bash
nano suricata.yaml
```
Add Custom Rule File -
Find section:
```yaml
rule-files
```
Add:
```yaml
- local.rules
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/1.png)

**Note:** Important Note
Check default rule path:
default-rule-path: /var/lib/suricata/rules

This is where Suricata loads custom rule definitions.

### Step 4 — Create Custom Suricata Rule
Go to rule directory:
```bash
cd /var/lib/suricata/rules
```
Create rule file:
```bash
nano local.rules
```
- Example Custom Rule (HTTP Detection):
```bash
alert http any any -> any any (
msg:"SOC ALERT: Suspicious HTTP User-Agent Detected";
http.user_agent;
content:"sqlmap";
nocase;
sid:1000003;
rev:1;
)
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/2.png)

Restart Suricata:
```bash
systemctl restart suricata
```
### Step 5 — Configure Wazuh Rules (DLL Injection Detection)
Navigate to Wazuh rules directory:
```bash
cd /var/ossec/etc/rules
ls
```
- Edit Local Rules File:
```bash
nano local_rules.xml
```
Add DLL Injection Detection Rule:
```xml
<group name="windows,sysmon">

  <rule id="100200" level="12">
    <if_sid>61610</if_sid>
    <description>Possible process injection activity detected from "$(win.eventdata.sourceImage)" on "$(win.eventdata.targetImage)"</description>
    <mitre>
      <id>T1055.001</id>
    </mitre>
  </rule>

  <rule id="100100" level="0">
    <if_sid>100200</if_sid>
    <field name="win.eventdata.sourceImage" type="pcre2">(C:\\\\Windows\\\\system32)|chrome.exe</field>
    <description>Ignore trusted Windows processes</description>
  </rule>

</group>
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/3.png)

Restart Wazuh Manager:
```bash
systemctl restart wazuh-manager
```
## Windows 10 Wazuh Agent Setup(Sysmon + DLL Injection Detection)
### Objective:
This section explains how to configure a Windows 10 endpoint for process injection (DLL Injection) detection using:
* Sysmon
* Wazuh
* Microsoft Windows

### Step 1 — Prepare Sysmon Files
Download Sysmon from Microsoft Sysinternals
Move the downloaded files into:
```powershell
C:\Sysmon\
```
### Step 2 — Edit Sysmon Configuration File
Open sysmonconfig.xml and ensure the following monitoring rules are included:
```xml
<ProcessCreate onmatch="include"/>
<ProcessAccess onmatch="include"/>
<CreateRemoteThread onmatch="include"/>
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/4.png)

### Purpose:
* Detect process creation events
* Detect process access (suspicious behavior)
* Detect DLL / process injection via remote thread creation

### Step 3 — Install Sysmon
Open PowerShell (Run as Administrator):
```powershell
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/5.png)

✔ This installs Sysmon with the custom configuration.

### Step 4 — Install Required Dependencies
Install:
Microsoft Visual C++ Redistributable (required for execution stability)

### Step 5 — Configure Wazuh Agent for Sysmon Logs
Navigate to Wazuh agent directory:
```text
C:\Program Files (x86)\ossec-agent
```
- Edit configuration file:
Open ossec.conf as Administrator and add:
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/6.png)

**Note:** Purpose: This enables Wazuh agent to collect Sysmon event logs from Windows Event Viewer.

### Step 6 — Restart Wazuh Agent
After configuration update, restart the agent:
```powershell
Restart-Service -Name wazuh
```
OR
```powershell
net stop wazuh
net start wazuh
```
### Step 7 — DLL Injection Simulation (Attack Test)
Run the injection tool in Administrator CMD:
```cmd
InjectProc.exe dll_inj hello-world-x64.dll cmd.exe
```
![image alt](https://github.com/knoxnz/SOC_Lab_Setup/blob/315bb21e44a6972dff75d2a839a8b0c863cf12d0/7.png)

### Expected Result:
* DLL injected into cmd.exe
* Sysmon logs generated (Event Viewer)
* Wazuh collects and forwards logs to SIEM

### Step 8 — Verification

Check logs in:

###  Windows Event Viewer
- Applications and Services Logs →
- Microsoft →
- Windows →
- Sysmon →
- Operational
### Wazuh Dashboard (Kibana)
- Alerts → Process Injection Detection
- Sysmon events

**Note** Reference Implementation
This setup is based on official Wazuh research: (https://wazuh.com/blog/detecting-process-injection-attacks-with-wazuh/) 

