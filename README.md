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
* Testing Firewall Policies and Web-Filtering on Alpine VM Clients
- Logs and Reports of Forwarding Traffic Captured on Fortinet Firewall

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
2. Insert the **NM-16ESW module** in the configure section and the slot to enable switch functionality.  
3. This converts the 3745 into an **EtherSwitch router** with 16 Ethernet switch ports, effectively functioning as the project’s LAN switch.


# Connecting Fortigate Firewall to Cloud Node to access internet

```
config system interface
    edit "port1"
        set vdom "root"
        set ip 192.168.1.2 255.255.255.0
        set allowaccess ping https ssh http
        set type physical
        set snmp-index 1
    next
end
```

### Connectivity to the internet

```
execute ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: icmp_seq=0 ttl=255 time=0.3 ms
64 bytes from 192.168.1.2: icmp_seq=1 ttl=255 time=0.0 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=255 time=0.0 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=255 time=0.0 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=255 time=0.0 ms

--- 192.168.1.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.0/0.0/0.3 ms

```

# Configuring Switch VLAN configuration 
```
ESW1#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ESW1(config)#interface range f1/0 - 3
ESW1(config-if-range)#switchport mode access
ESW1(config-if-range)#
ESW1(config-if-range)#switchport access vlan 1

```

<img width="836" height="260" alt="image" src="https://github.com/user-attachments/assets/879f9094-35a6-4164-bfbb-9711cb888a0c" />



# DHCP Server configurations on the Fortigate Firewall

```
config system dhcp server
    edit 1
        set lease-time 86400
        set default-gateway 192.168.10.1
        set netmask 255.255.255.0
        set interface "port2"
        config ip-range
            edit 1
                set start-ip 192.168.10.10
                set end-ip 192.168.10.100
            next
        end
        set dns-server1 8.8.8.8
        set dns-server2 8.8.4.4
    next
end

```

# Alpine Virtual Machines getting IP Addresses dynamically through the DHCP server configured on the Fortinet Firewall

```
alpine:~# udhcpc -v eth0
```
```
udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: broadcasting select for 192.168.10.10, server 192.168.10.1
udhcpc: lease of 192.168.10.10 obtained from 192.168.10.1, lease time 86400
```

# Firewall Addressing Configuration (Web-filtering : Block access to "Facebook" and "YouTube")

```
config firewall address
edit "facebook"
        set subnet 157.240.0.0 255.255.0.0
    next
    edit "youtube"
        set type fqdn
        set fqdn "youtube.com"
    next
    edit "ytcdn"
        set type fqdn
        set fqdn "*.googlevideo.com"
    next
end

```

# Firewall Policy Configuration 

```
config firewall policy
    edit 200
        set name "Block Facebook"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "facebook"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 300
        set name "Block Youtube"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "youtube" "ytcdn"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 100
        set name "LAN-WAN NAT"
        set srcintf "port2"
        set dstintf "port1"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set nat enable
    next
end

```

# Testing Firewall Policies and Web-Filtering on Alpine VM Clients

```
alpine:~# ping facebook.com
```

```
PING facebook.com (157.240.214.35): 56 data bytes
^C
--- facebook.com ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```


```
alpine:~# ping bbc.com
```

```
PING bbc.com (151.101.192.81): 56 data bytes
64 bytes from 151.101.192.81: seq=0 ttl=58 time=20.265 ms
64 bytes from 151.101.192.81: seq=1 ttl=58 time=16.056 ms
64 bytes from 151.101.192.81: seq=2 ttl=58 time=28.984 ms
^C
--- bbc.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
```

```
ping Youtube.com
```

```
PING Youtube.com (142.250.129.190): 56 data bytes
^C
--- Youtube.com ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

```
alpine:~# ping google.com
```

```
PING google.com (142.250.179.238): 56 data bytes
64 bytes from 142.250.179.238: seq=0 ttl=116 time=17.773 ms
64 bytes from 142.250.179.238: seq=1 ttl=116 time=32.817 ms
64 bytes from 142.250.179.238: seq=2 ttl=116 time=20.215 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 17.773/23.601/32.817 ms
```

# Logs and Reports of Forwarding Traffic Captured 

<img width="1916" height="910" alt="image" src="https://github.com/user-attachments/assets/499866f0-bd04-4970-958f-2342cb1b9655" />



