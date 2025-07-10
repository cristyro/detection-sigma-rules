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
- On UTM you can download the prebuilt images. Just click on UTM from the zip (unextracted)
then run all the following commands :
#add explanation what each one is for pls
sudo apt update && sudo apt upgrade -y
sudo apt install curl apt-transport-https gnupg2 -y
then 
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg arch=arm64] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
then
sudo apt update
sudo apt install wazuh-manager -y
then 
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-manager
then
sudo systemctl status wazuh-manager 


once it is done 
allow all (open firewall to alllow )
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp 
1514: agents send logs
1515: for agent registration

to note the wazuh manager 
ip a 
then use this ip adress to put on the windows agent ossec.conf <address>DEBIAN_VM_IP</address> 

--- actually maybe put all this above in one bash in some part of the library? reference to it here but do explain what 
each one is for 



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