# 🛡️ Detection Lab Setup Guide

This guide documents how I built a personal detection lab for testing and writing Sigma detection rules.

---

##  **Goal**
- Simulate real-world attacks in a controlled environment
- Collect logs with a SIEM (Wazuh)
- Write and test custom Sigma detection rules

---

##  **Lab Architecture**

![Lab Diagram](diagram.png)

| VM | OS | Purpose |
| Kali Linux | Debian-based | Attacker machine to run simulated attacks |
| Windows 10/11 Pro | Windows | Victim machine, sends logs to SIEM |
| Debian/Ubuntu | Linux | SIEM server (Wazuh) |

All VMs run locally inside **UTM** on macOS.

---

##  **Tools & stack**
-  **Wazuh**: open-source SIEM, includes ELK stack and dashboards
-  **Sysmon**: detailed Windows event logging
-  **Atomic Red Team**: simulate attacker techniques
-  **Sigma**: rule format to detect attacks in logs
-  **UTM**: virtual machine manager for macOS

---

##  **Step-by-step setup**

###  1. Install UTM
- Download: [https://mac.getutm.app/](https://mac.getutm.app/)
- Create a Projects folder: `~/Projects`

---

###  2. Create VMs
- Change the VM architecture to local computer's architecture 

####  Windows 10/11 Pro (Victim)
This machine acts as the targeted endpoint.

**Steps:**

- Download a Windows ARM64 ISO  
  - You can use **Crystal Fetch** or **UUP Dump** to get the latest build
- Create the VM in UTM using the ISO
- During boot, press any key if screen stays black
- Install **Windows 11 Pro** or the latest version
- After setup, delete the ISO from the virtual drive to avoid rebooting into installation

**Install Sysmon:**

- Use [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- Suggested config: [SwiftOnSecurity Sysmon config](https://github.com/SwiftOnSecurity/sysmon-config)
- Purpose: Records detailed Windows logs like process creation, network connections, and file events

> 🔧 Use the Microsoft Store or download the Sysinternals ZIP manually

Example tutorial link (search-based):  
[Sysmon setup guide](https://www.google.com/search?q=sysmon&oq=sysmon#fpstate=ive&vld=cid:654d40a7,vid:98B9UmFr0qs,st:0)

**Install Wazuh Agent:**

- Wazuh Agent path:  
  `C:\Program Files (x86)\ossec-agent`
- After Wazuh Manager is ready (on the Debian VM), link to its IP address
- You'll configure the agent in `ossec.conf`:
  ```xml
  <address>DEBIAN_VM_IP</address>
  ```

**Enable Logging Policies (TODO):**

- Enable PowerShell logging
- Enable process auditing via Local Group Policy  
  *(Instructions pending)*

####  SIEM Server (Debian/Ubuntu)
Use **Ubuntu Server 22.04** (or similar) for compatibility with OpenSearch and Wazuh. This server will act as your **SIEM**, collecting logs from all endpoints and making them accessible via a web dashboard.
**Key goals:**
- Headless SIEM server
- Wazuh Dashboard accessible via browser
- OpenSearch backend for log storage
- Central log receiver (Windows / Kali / Linux)
### 🔧 Wazuh Manager Installation
**Update and install required packages:**
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg arch=arm64] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```
**Add Wazuh GPG key and repository:**
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg arch=arm64] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```
**Install Wazuh Manager:**
```bash
sudo apt update
sudo apt install wazuh-manager -y
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-manager
sudo systemctl status wazuh-manager
```
### Install OpenSearch and Dashboard
**Add OpenSearch GPG key and repo:**
```bash
curl -fsSL https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo gpg --dearmor -o /usr/share/keyrings/opensearch-keyring
echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring arch=arm64] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch.list
```
**Install OpenSearch and dependencies:**
```bash
sudo apt install openjdk-17-jdk opensearch -y
```
> ⚠️ You must set a strong password or installation will fail.
**Run security initialization script:**
> Note : replace MyStrongAdminPass123! with your chosen password
```bash
cd /usr/share/opensearch/plugins/opensearch-security/tools/
sudo OPENSEARCH_INITIAL_ADMIN_PASSWORD="MyStrongAdminPass123!" bash ./install_demo_configuration.sh

**Configure Firewall Rules **
sudo ufw allow 1514/tcp   # Wazuh agent logs
sudo ufw allow 1515/tcp   # Wazuh agent registration
sudo ufw allow 5601/tcp   # Wazuh dashboard (Kibana equivalent)
** Check for your ip adress** 
** Update your windows agent ossec.conf file with the IP adress you obtained **
```
<address>DEBIAN_VM_IP</address>
```

** Verify everything works by running **

