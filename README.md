# Air-Gapped SOC & Active Defense Lab

![soc_lab](https://github.com/user-attachments/assets/1626b5f0-8696-4b5b-b6a4-cb5c2250f96c)


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

![eth1](https://github.com/user-attachments/assets/0df55331-474a-45c7-b7e0-cdc23db00185)

![eth2](https://github.com/user-attachments/assets/cd71d4ea-9d07-4b52-8741-58191a5999ff)


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

![kalinmap](https://github.com/user-attachments/assets/219734aa-5a34-4aca-84fd-b07f282ead36)


* Executed a command to clear the Windows Security logs (`wevtutil cl security`) to simulate an attacker attempting Defense Evasion.

### Step 5: Blue Team Detection & Analysis
* The Wazuh Agent successfully intercepted the Sysmon telemetry and forwarded it to the Manager.
* The SIEM correctly flagged the activity, categorizing it under the MITRE ATT&CK framework:
  * **Discovery (T1046):** Network Service Discovery (The Nmap scan)
  * **Defense Evasion (T1070):** Indicator Removal on Host (The cleared logs)

![wazuh](https://github.com/user-attachments/assets/8f8cb195-b01c-4918-a9ff-60800dca6111)

## Conclusion
This lab provided hands-on experience in bridging virtualized networking with physical hardware, configuring endpoint detection mechanisms, and translating raw system logs into actionable security alerts.
