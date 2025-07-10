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
poor endpoint being attacked 
- Download Windows ARM64 ISO 
   notes on this
   I used Crystal Fetch to produce the latest ISO 
   Once generated, create the vm using this existing iso 
   - boot up by pressing any key on black screen. 
   - install windows 11 pro (or latest release )
   - make sure to delete the iso after the installation to avoid booting to the installation again idk why this happens 
- Install Sysmon with a standard config (e.g., SwiftOnSecurity)
   Sysmon is a free logging tool in windows we are going to use to record the logs of the attacks and check procceses
   (install from microsoft store - sysinternalsuite )
   -> link to tuto - https://www.google.com/search?q=sysmon&oq=sysmon&gs_lcrp=EgZjaHJvbWUyCQgAEEUYORiABDIHCAEQABiABDIHCAIQABiABDIHCAMQABiABDIHCAQQABiABDIHCAUQABiABDIHCAYQABiABDIHCAcQABiABDIHCAgQABiABDIHCAkQABiABNIBCDE1MDhqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8#fpstate=ive&vld=cid:654d40a7,vid:98B9UmFr0qs,st:0

- Install Wazuh agent: 
C:\Program Files (x86)\ossec-agent this is where wazuh wizard is saved according to wazuh 
  - Link to Wazuh manager IP 
  Note to do this have to have debian machine setup as this is the wazuh manager
  todo still need to comment in this 
- Enable PowerShell & process auditing in local group policy (more on this!! #todo)

####  SIEM Server (Debian/Ubuntu)

This is the wazuh manager/ siem server. This is where we collect all the logs from the windows vm like 
a sys admin seeing an endpoint affected 
- Download latest ARM64 Ubuntu/Debian ISO
- Install Wazuh:
  - Use Docker quickstart:
    ```bash
    curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
    sudo bash ./wazuh-install.sh -a
    ```
- Expose Wazuh web UI on local port

####  Kali Linux (Attacker)
This is the attacker trying to get in and sending traffick
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

###  4. Simulate attacks
- Use Atomic Red Team:
  ```bash
  Invoke-AtomicTest T1059.001