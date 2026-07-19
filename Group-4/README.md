# Group‑4 Configuration Steps

## 📝 Task Flow
1. Initial configuration on SW‑53  
2. Create VLANs (50, 60, and VLAN 1 for management)  
3. Configure trunk port on SW‑53  
4. Configure R11 as Router on a Stick  
5. Configure R11 as DHCP server  
6. Initial configuration on R10, R11, R12  
7. Configure serial interfaces on all routers  
8. Configure OSPF routing on all routers (Area 0)  
9. Configure PAT on R11  
10. Configure Extended ACL on R10 to block HTTP access to R12 LAN  

## Lab Topology
![Group-4 Lab](Group-4.gif)

---
## 🔧 Device IP Table
| Device | F0/0        | S0/0       | S0/1       |
|--------|-------------|------------|------------|
| R10    | 192.168.9.1 | 172.25.0.1 | -          |
| R11    | 10.0.0.3    | 172.26.0.1 | 172.25.0.2 |
| R12    | 192.168.10.1| -          | 172.26.0.2 |
| SW‑53  | VLAN 1 → 192.168.23.53 | - | - |
---

## ⚙️ Configurations

### Switch SW‑53

Switch> enable  
Switch# configure terminal  
Switch(config)# hostname SW-53  

**VLAN 1 Management IP**  
SW-53(config)# interface vlan 1  
SW-53(config-if)# ip address 192.168.23.53 255.255.255.0  
SW-53(config-if)# no shutdown  

**Vlans Configuration**  
SW-53(config)# vlan 50  
SW-53(config-vlan)# name VLAN50  
SW-53(config)# vlan 60  
SW-53(config-vlan)# name VLAN60  

**Trunk Configuration**  
SW-53(config)# interface fa0/23  
SW-53(config-if)# switchport mode trunk  

---

## R10 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R10  
R10(config)# no ip domain-lookup  
R10(config)# interface fa0/0  
R10(config-if)# ip address 192.168.9.1 255.255.255.0  
R10(config-if)# no shutdown  
R10(config)# interface s0/0  
R10(config-if)# ip address 172.25.0.1 255.255.255.0  
R10(config-if)# clock rate 64000  
R10(config-if)# no shutdown  

**OSPF Routing**  
R10(config)# router ospf 1  
R10(config-router)# network 192.168.9.0 0.0.0.255 area 0  
R10(config-router)# network 172.25.0.0 0.0.0.255 area 0  

**Extended ACL (Block HTTP to R12 LAN)**  
R10(config)# access-list 101 deny tcp any 192.168.10.0 0.0.0.255 eq 80  
R10(config)# access-list 101 permit ip any any  
R10(config)# interface fa0/0  
R10(config-if)# ip access-group 101 in  

---

## R11 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R11  
R11(config)# no ip domain-lookup  

**Router on a Stick**  
R11(config)# interface fa0/0.1  
R11(config-subif)# encapsulation dot1Q 50  
R11(config-subif)# ip address 192.168.50.1 255.255.255.0  

R11(config)# interface fa0/0.2  
R11(config-subif)# encapsulation dot1Q 60  
R11(config-subif)# ip address 192.168.60.1 255.255.255.0  

**DHCP Pools**  
R11(config)# ip dhcp pool VLAN50  
R11(dhcp-config)# network 192.168.50.0 255.255.255.0  
R11(dhcp-config)# default-router 192.168.50.1  

R11(config)# ip dhcp pool VLAN60  
R11(dhcp-config)# network 192.168.60.0 255.255.255.0  
R11(dhcp-config)# default-router 192.168.60.1  

**Serial Interfaces configuration**  
R11(config)# interface s0/1  
R11(config-if)# ip address 172.25.0.2 255.255.255.0  
R11(config-if)# no shutdown  

R11(config)# interface s0/0  
R11(config-if)# ip address 172.26.0.1 255.255.255.0  
R11(config-if)# clock rate 64000  
R11(config-if)# no shutdown  

**OSPF Routing**  
R11(config)# router ospf 1  
R11(config-router)# network 10.0.0.0 0.0.0.255 area 0  
R11(config-router)# network 172.25.0.0 0.0.0.255 area 0  
R11(config-router)# network 172.26.0.0 0.0.0.255 area 0  
R11(config-router)# network 192.168.50.0 0.0.0.255 area 0  
R11(config-router)# network 192.168.60.0 0.0.0.255 area 0  

**PAT Configuration**  
R11(config)# access-list 1 permit 192.168.50.0 0.0.0.255  
R11(config)# access-list 1 permit 192.168.60.0 0.0.0.255  
R11(config)# ip nat inside source list 1 interface fa0/0 overload  
R11(config)# interface fa0/0.1  
R11(config-if)# ip nat inside  
R11(config)# interface fa0/0.2  
R11(config-if)# ip nat inside  
R11(config)# interface fa0/0  
R11(config-if)# ip nat outside  

---

## R12 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R12  
R12(config)# no ip domain-lookup  
R12(config)# interface fa0/0  
R12(config-if)# ip address 192.168.10.1 255.255.255.0  
R12(config-if)# no shutdown  
R12(config)# interface s0/1  
R12(config-if)# ip address 172.26.0.2 255.255.255.0  
R12(config-if)# no shutdown  

**OSPF Routing**  
R12(config)# router ospf 1  
R12(config-router)# network 192.168.10.0 0.0.0.255 area 0  
R12(config-router)# network 172.26.0.0 0.0.0.255 area 0  

---

## Verification
SW-53# show vlan brief → VLANs active  

SW-53# ping 192.168.23.53 → Management IP reachable  

R10# show ip route  
R11# show ip route  
R12# show ip route → OSPF routes learned  

R11# show ip nat translations → PAT working  

R10# show access-lists → Extended ACL applied  

PC> ping <destination IP> → End‑to‑end connectivity  

PC> telnet <R12 LAN IP> 80 → Blocked by ACL  

**Note: HTTP traffic from any source to R12 LAN is denied due to Extended ACL.**


## 📌 Final Notes

- Always save configurations with `write memory` or `copy running-config startup-config`.  
- Use `ping` and `traceroute` to verify connectivity across VLANs and routers.  
- Check routing tables with `show ip route` to confirm static, RIP, EIGRP, or OSPF entries.  
- Validate ACLs with `show access-lists` and NAT/PAT with `show ip nat translations`.  
- Document any changes made during lab practice for future reference.  

## 🎯 Learning Outcomes

By completing this lab, you will:
- Strengthen skills in VLANs, trunking, and Router on a Stick.  
- Practice DHCP configuration for dynamic IP allocation.  
- Implement and verify routing protocols (Static, RIP, EIGRP, OSPF).  
- Apply ACLs for traffic filtering and service control.  
- Configure NAT/PAT for address translation.  
- Build confidence in troubleshooting and verification commands.  

---
