# Air-Gapped SOC & Active Defense Lab

<img width="951" height="773" alt="soc_lab" src="https://github.com/user-attachments/assets/7140ba6e-87e9-47e3-a56e-3f812573b846" />


## Project Overview
This project simulates a complete, end-to-end cyber attack and detection lifecycle. To ensure safety and zero leakage to production networks, I engineered a fully air-gapped physical network bridging an attacker virtual machine with a physical Windows target. I then deployed an enterprise-grade SIEM to detect and map attacks to the MITRE ATT&CK framework.

## Architecture & Tech Stack
* **Attacker System:** Kali Linux (VM via UTM on macOS)
* **Target System:** Windows 11 Desktop (Physical Host)
* **SIEM / XDR:** Wazuh Manager (VirtualBox OVA)
* **Endpoint Telemetry:** Microsoft Sysmon
* **Networking:** Direct Ethernet bridging with static IP routing (`192.168.100.x`), Promiscuous Mode enabled.

## Step-by-Step Execution

### Step 1: Network Isolation & Infrastructure
* Disabled all physical Wi-Fi adapters to create a true air-gapped environment.
* Established a direct physical connection via Ethernet between the macOS host and Windows target.
* Assigned static IP addresses to the virtual and physical interfaces to ensure localized routing.

<img width="1034" height="773" alt="eth1" src="https://github.com/user-attachments/assets/47f51f47-0dd3-4340-b67f-6f0487c3e76d" />


<img width="1037" height="777" alt="eth2" src="https://github.com/user-attachments/assets/8bfc619d-8edd-468d-97f1-e1ffaa530672" />


### Step 2: Target Telemetry Configuration
* Installed **Microsoft Sysmon** on the Windows 11 target.
* Configured Sysmon to log network connections (Event ID 3) to capture inbound reconnaissance.
* Deployed a Python HTTP listener (`python -m http.server 8080`) to act as a vulnerable service target.

### Step 3: SIEM Deployment
* Deployed the **Wazuh Manager** via VirtualBox on the Windows host.
* Configured the VirtualBox Bridged Network Adapter to monitor the physical Ethernet connection.
* Installed the Wazuh Agent on the Windows target and modified the `ossec.conf` file to ingest Sysmon Operational logs.

### Step 4: Red Team Execution
* Initiated a targeted, aggressive Nmap scan from the Kali Linux VM across the physical cable targeting the open Python port:
  `sudo nmap -sV -p 8080 192.168.100.2`

<img width="1034" height="771" alt="kalinmap" src="https://github.com/user-attachments/assets/b2d5046c-7697-4399-a229-c34a267b81bf" />


* Executed a command to clear the Windows Security logs (`wevtutil cl security`) to simulate an attacker attempting Defense Evasion.

### Step 5: Blue Team Detection & Analysis
* The Wazuh Agent successfully intercepted the Sysmon telemetry and forwarded it to the Manager.
* The SIEM correctly flagged the activity, categorizing it under the MITRE ATT&CK framework:
  * **Discovery (T1046):** Network Service Discovery (The Nmap scan)
  * **Defense Evasion (T1070):** Indicator Removal on Host (The cleared logs)

<img width="1035" height="723" alt="wazuh" src="https://github.com/user-attachments/assets/83a06910-23da-43c7-9f04-ccbad92cc65b" />


## Conclusion
This lab provided hands-on experience in bridging virtualized networking with physical hardware, configuring endpoint detection mechanisms, and translating raw system logs into actionable security alerts.
