# Group‑3 Configuration Steps

## 📝 Task Flow
1. Initial configuration on SW‑52  
2. Create VLANs (50, 60, and VLAN 1 for management)  
3. Configure Fa0/23 as trunk on SW‑52  
4. Configure R5 as Router on a Stick  
5. Configure R5 as DHCP server  
6. Initial configuration on R4, R5, R6  
7. Configure serial interfaces on all routers  
8. Configure EIGRP routing on all routers (AS 100)  
9. Configure Static NAT on R5  

## Lab Topology
![Group-3 Lab](Group-3.gif)

---
## 🔧 Device IP Table
| Device | F0/0        | S0/0       | S0/1       |
|--------|-------------|------------|------------|
| R4     | 192.168.4.1 | 172.19.0.1 | -          |
| R5     | 10.0.0.2    | 172.20.0.1 | 172.19.0.2 |
| R6     | 192.168.5.1 | -          | 172.20.0.2 |
| SW‑52  | VLAN 1 → 192.168.22.52 | - | - |
---

## ⚙️ Configurations

### Switch SW‑52

Switch> enable  
Switch# configure terminal  
Switch(config)# hostname SW-52  

**VLAN 1 Management IP**  
SW-52(config)# interface vlan 1  
SW-52(config-if)# ip address 192.168.22.52 255.255.255.0  
SW-52(config-if)# no shutdown  

**Vlans Configuration**  
SW-52(config)# vlan 50  
SW-52(config-vlan)# name VLAN50  
SW-52(config)# vlan 60  
SW-52(config-vlan)# name VLAN60  

**Trunk Configuration**  
SW-52(config)# interface fa0/23  
SW-52(config-if)# switchport mode trunk  

---

## R4 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R4  
R4(config)# no ip domain-lookup  
R4(config)# interface fa0/0  
R4(config-if)# ip address 192.168.4.1 255.255.255.0  
R4(config-if)# no shutdown  
R4(config)# interface s0/0  
R4(config-if)# ip address 172.19.0.1 255.255.255.0  
R4(config-if)# clock rate 64000  
R4(config-if)# no shutdown  

**EIGRP Routing**  
R4(config)# router eigrp 100  
R4(config-router)# network 192.168.4.0 0.0.0.255  
R4(config-router)# network 172.19.0.0 0.0.0.255  

---

## R5 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R5  
R5(config)# no ip domain-lookup  

**Router on a Stick**  
R5(config)# interface fa0/0.1  
R5(config-subif)# encapsulation dot1Q 50  
R5(config-subif)# ip address 192.168.50.1 255.255.255.0  

R5(config)# interface fa0/0.2  
R5(config-subif)# encapsulation dot1Q 60  
R5(config-subif)# ip address 192.168.60.1 255.255.255.0  

**DHCP Pools**  
R5(config)# ip dhcp pool VLAN50  
R5(dhcp-config)# network 192.168.50.0 255.255.255.0  
R5(dhcp-config)# default-router 192.168.50.1  

R5(config)# ip dhcp pool VLAN60  
R5(dhcp-config)# network 192.168.60.0 255.255.255.0  
R5(dhcp-config)# default-router 192.168.60.1  

**Serial Interfaces configuration**  
R5(config)# interface s0/1  
R5(config-if)# ip address 172.19.0.2 255.255.255.0  
R5(config-if)# no shutdown  

R5(config)# interface s0/0  
R5(config-if)# ip address 172.20.0.1 255.255.255.0  
R5(config-if)# clock rate 64000  
R5(config-if)# no shutdown  

**EIGRP Routing**  
R5(config)# router eigrp 100  
R5(config-router)# network 10.0.0.0 0.0.0.255  
R5(config-router)# network 172.19.0.0 0.0.0.255  
R5(config-router)# network 172.20.0.0 0.0.0.255  
R5(config-router)# network 192.168.50.0 0.0.0.255  
R5(config-router)# network 192.168.60.0 0.0.0.255  

**Static NAT**  
R5(config)# ip nat inside source static 192.168.50.10 10.0.0.100  
R5(config)# interface fa0/0.1  
R5(config-if)# ip nat inside  
R5(config)# interface fa0/0.2  
R5(config-if)# ip nat inside  
R5(config)# interface fa0/0  
R5(config-if)# ip nat outside  

---

## R6 Router Configuration

Router> enable  
Router# configure terminal  
Router(config)# hostname R6  
R6(config)# no ip domain-lookup  
R6(config)# interface fa0/0  
R6(config-if)# ip address 192.168.5.1 255.255.255.0  
R6(config-if)# no shutdown  
R6(config)# interface s0/1  
R6(config-if)# ip address 172.20.0.2 255.255.255.0  
R6(config-if)# no shutdown  

**EIGRP Routing**  
R6(config)# router eigrp 100  
R6(config-router)# network 192.168.5.0 0.0.0.255  
R6(config-router)# network 172.20.0.0 0.0.0.255  

---

## Verification
SW-52# show vlan brief → VLANs active  

SW-52# ping 192.168.22.52 → Management IP reachable  

R4# show ip route  
R5# show ip route  
R6# show ip route → EIGRP routes learned  

R5# show ip nat translations → NAT working  

PC> ping <destination IP> → End‑to‑end connectivity  

**Note: NAT ensures internal VLAN hosts can access external networks using translated addresses.**

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
