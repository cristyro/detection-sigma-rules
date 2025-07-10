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
- Download Windows ARM64 ISO (Insider Preview)
- Install Sysmon with a standard config (e.g., SwiftOnSecurity)
- Install Wazuh agent:
  - Link to Wazuh manager IP
- Enable PowerShell & process auditing in local group policy (more on this!! #todo)

####  SIEM VM (Debian/Ubuntu)
- Download latest ARM64 Ubuntu/Debian ISO
- Install Wazuh:
  - Use Docker quickstart:
    ```bash
    curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
    sudo bash ./wazuh-install.sh -a
    ```
- Expose Wazuh web UI on local port

####  Kali Linux (Attacker)
- Download Kali ARM64 ISO
- Install tools:
  - `git`, `python3`, Atomic Red Team

---

###  3. Configure networking
- Use **shared networking / bridged mode** in UTM so all VMs can see each other
- Confirm:
  - Windows can ping SIEM server
  - SIEM server receives logs from agent

---

### ✅ 4. Simulate attacks
- Use Atomic Red Team:
  ```bash
  Invoke-AtomicTest T1059.001