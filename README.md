# Secure-Internet-Gateway-Project

Built a secure Internet gateway by integrating Cisco LAN with a FortiGate firewall. Configured NAT, firewall policies, and manual URL/IP filtering, then validated with Linux client testing to demonstrate routing, firewall, and network security skills.

Table of contents
- Overview
* Network Topology
+ IP Addressing
- Networking Devices Setup
* Connecting Fortigate Firewall to Cloud node to access Internet
+ Switch VLAN Configuration 
- DHCP Server Configuration
* Alpine Linux Clients dynamically getting IP addresses
+ Firewall Addressing Configuration (Web-filtering : Block access to "Facebook" and "YouTube")
- Firewall Policy Configuration

# Network Topology
<img width="1002" height="830" alt="image" src="https://github.com/user-attachments/assets/b19746fc-328b-4d64-ae8f-c4138ebb3c59" />

# IP Addressing Scheme

| Device           | Interface          | IP Address          | Role                         |
|------------------|--------------------|-------------------- |-------------------------------|
| FortiGate        | port1 (WAN side)   | 192.168.1.2/24      | Outbound to internet          |
| FortiGate        | port2 (LAN side)   | 192.168.10.1        | Inside firewall interface     |
| Cisco 3745       | NM-16ESW ports     | VLAN 1 (switching)  | Access switch for LAN PCs     |
| Alpine VMs       | eth0               | DHCP assignments    | Client for testing            |


# Adding the Cisco 3745 as an EtherSwitch
1. Add the **Cisco 3745** router to the worksheet   
2. Insert the **NM-16ESW module** in the configure template section to enable switch functionality.  
3. This converts the 3745 into an **EtherSwitch router** with 16 Ethernet switch ports, effectively functioning as the project’s LAN switch.


# Connecting Fortigate Firewall to Cloud Node to access internet




