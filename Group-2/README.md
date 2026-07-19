# Group‑2 Configuration Steps

## 📝 Task Flow
1. Initial configuration on SW‑51  
2. Create VLANs (30, 40, and VLAN 1 for management)  
3. Configure Fa0/24 as trunk on SW‑51  
4. Configure R8 as Router on a Stick  
5. Configure R8 as DHCP server  
6. Initial configuration on R7, R8, R9  
7. Configure serial interfaces on all routers  
8. Configure RIP routing on all routers  
9. Configure Extended ACL on R7 to block HTTP access to R9 LAN  

## Lab Topology
![Group-2 Lab](Group-2.gif)

---
## 🔧 Device IP Table
| Device | F0/0        | S0/0       | S0/1       |
|--------|-------------|------------|------------|
| R7     | 192.168.6.1 | 172.22.0.1 | -          |
| R8     | 10.0.0.3    | 172.23.0.1 | 172.22.0.2 |
| R9     | 192.168.7.1 | -          | 172.23.0.2 |
| SW‑51  | VLAN 1 → 192.168.20.51 | - | - |
---

## ⚙️ Configurations

### Switch SW‑51

Switch> enable  
Switch# configure terminal  
Switch(config)# hostname SW-51  

**VLAN 1 Management IP**  
SW-51(config)# interface vlan 1  
SW-51(config-if)# ip address 192.168.20.51 255.255.255.0  
SW-51(config-if)# no shutdown  

**Vlans Configuration**  
SW-51(config)# vlan 30  
SW-51(config-vlan)# name VLAN30  
SW-51(config)# vlan 40  
SW-51(config-vlan)# name VLAN40  

**Trunk Configuration**  
SW-51(config)# interface fa0/24  
SW-51(config-if)# switchport mode trunk  

---

## R7 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R7  
R7(config)# no ip domain-lookup  
R7(config)# interface fa0/0  
R7(config-if)# ip address 192.168.6.1 255.255.255.0  
R7(config-if)# no shutdown  
R7(config)# interface s0/0  
R7(config-if)# ip address 172.22.0.1 255.255.255.0  
R7(config-if)# clock rate 64000  
R7(config-if)# no shutdown  

**RIP Routing**  
R7(config)# router rip  
R7(config-router)# version 2  
R7(config-router)# network 192.168.6.0  
R7(config-router)# network 172.22.0.0  

**Extended ACL (Block HTTP to R9 LAN)**  
R7(config)# access-list 100 deny tcp any 192.168.7.0 0.0.0.255 eq 80  
R7(config)# access-list 100 permit ip any any  
R7(config)# interface fa0/0  
R7(config-if)# ip access-group 100 in  

---

## R8 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R8  
R8(config)# no ip domain-lookup  

**Router on a Stick**  
R8(config)# interface fa0/0.1  
R8(config-subif)# encapsulation dot1Q 30  
R8(config-subif)# ip address 192.168.30.1 255.255.255.0  

R8(config)# interface fa0/0.2  
R8(config-subif)# encapsulation dot1Q 40  
R8(config-subif)# ip address 192.168.40.1 255.255.255.0  

**DHCP Pools**  
R8(config)# ip dhcp pool VLAN30  
R8(dhcp-config)# network 192.168.30.0 255.255.255.0  
R8(dhcp-config)# default-router 192.168.30.1  

R8(config)# ip dhcp pool VLAN40  
R8(dhcp-config)# network 192.168.40.0 255.255.255.0  
R8(dhcp-config)# default-router 192.168.40.1  

**Serial Interfaces configuration**  
R8(config)# interface s0/1  
R8(config-if)# ip address 172.22.0.2 255.255.255.0  
R8(config-if)# no shutdown  

R8(config)# interface s0/0  
R8(config-if)# ip address 172.23.0.1 255.255.255.0  
R8(config-if)# clock rate 64000  
R8(config-if)# no shutdown  

**RIP Routing**  
R8(config)# router rip  
R8(config-router)# version 2  
R8(config-router)# network 10.0.0.0  
R8(config-router)# network 172.22.0.0  
R8(config-router)# network 172.23.0.0  
R8(config-router)# network 192.168.30.0  
R8(config-router)# network 192.168.40.0  

---

## R9 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R9  
R9(config)# no ip domain-lookup  
R9(config)# interface fa0/0  
R9(config-if)# ip address 192.168.7.1 255.255.255.0  
R9(config-if)# no shutdown  
R9(config)# interface s0/1  
R9(config-if)# ip address 172.23.0.2 255.255.255.0  
R9(config-if)# no shutdown  

**RIP Routing**  
R9(config)# router rip  
R9(config-router)# version 2  
R9(config-router)# network 192.168.7.0  
R9(config-router)# network 172.23.0.0  

---

## Verification
SW-51# show vlan brief → VLANs active  

SW-51# ping 192.168.21.51 → Management IP reachable  

R7# show ip route  
R8# show ip route  
R9# show ip route → RIP routes learned  

R7# show access-lists → Extended ACL applied  

PC> ping <destination IP> → End‑to‑end connectivity  

PC> telnet <R9 LAN IP> 80 → Blocked by ACL  

**Note: HTTP traffic from any source to R9 LAN is denied due to Extended ACL.**

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
