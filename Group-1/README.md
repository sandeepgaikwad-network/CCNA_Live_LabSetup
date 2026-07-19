    # Group‑1 Configuration Steps

    ## 📝 Task Flow
    1. Initial configuration on SW‑50  
    2. Create VLANs (30, 40, and VLAN 1 for management)  
    3. Configure Fa0/24 as trunk on SW‑50  
    4. Configure R2 as Router on a Stick  
    5. Configure R2 as DHCP server  
    6. Initial configuration on R1, R2, R3  
    7. Configure serial interfaces on all routers  
    8. Configure static routes on all routers  
    9. Configure standard ACL on R1  

    ## Lab Topology
    ![Group-1 Lab](Group-1.gif)

    ---
    ## 🔧 Device IP Table
    | Device | F0/0        | S0/0       | S0/1       |
    |--------|-------------|------------|------------|
    | R1     | 192.168.2.1 | 172.16.0.1 | -          |
    | R2     | 10.0.0.1    | 172.17.0.1 | 172.16.0.2 |
    | R3     | 192.168.3.1 | -          | 172.17.0.2 |
    | SW‑50  | VLAN 1 → 192.168.20.50 | - | - |
    ---

    ## ⚙️ Configurations

    ### Switch SW‑50

    Switch> enable  
    Switch# configure terminal  
    Switch0(config)# hostname SW-50  

    **VLAN 1 Management IP**  
    SW-50(config)# interface vlan 1  
    SW-50(config-if)# ip address 192.168.20.50 255.255.255.0  
    SW-50(config-if)# no shutdown

    **Vlans Configuration**  
    SW-50(config)# vlan 30  
    SW-50(config-vlan)# name VLAN30  
    SW-50(config)# vlan 40  
    SW-50(config-vlan)# name VLAN40  

    **Trunk Configuration**  
    SW-50(config)# interface fa0/24  
    SW-50(config-if)# switchport mode trunk

    ## R1 Router Configuration Inital Configuration

    Router> enable  
    Router# configure terminal  
    Router(config)# hostname R1  
    R1(config)# no ip domain-lookup  
    R1(config)# interface fa0/0  
    R1(config-if)# ip address 192.168.2.1 255.255.255.0  
    R1(config-if)# no shutdown  
    R1(config)# interface s0/0  
    R1(config-if)# ip address 172.16.0.1 255.255.255.0  
    R1(config-if)# clock rate 64000  
    R1(config-if)# no shutdown  

    **Static Routing**  
    R1(config)# ip route 192.168.3.0 255.255.255.0 172.16.0.2

    **Standard Access-Control List**  
    R1(config)# access-list 1 deny 192.168.3.0 0.0.0.255  
    R1(config)# access-list 1 permit any  
    R1(config)# interface fa0/0  
    R1(config-if)# ip access-group 1 in

    ## R2 Router Configuration

    Router> enable  
    Router# configure terminal  
    Router(config)# hostname R2  
    R2(config)# no ip domain-lookup  

    **Router on a Stick**  
    R2(config)# interface fa0/0.1  
    R2(config-subif)# encapsulation dot1Q 30  
    R2(config-subif)# ip address 192.168.30.1 255.255.255.0

    R2(config)# interface fa0/0.2  
    R2(config-subif)# encapsulation dot1Q 40  
    R2(config-subif)# ip address 192.168.40.1 255.255.255.0

    **DHCP Pools**  
    R2(config)# ip dhcp pool VLAN30  
    R2(dhcp-config)# network 192.168.30.0 255.255.255.0  
    R2(dhcp-config)# default-router 192.168.30.1  

    R2(config)# ip dhcp pool VLAN40  
    R2(dhcp-config)# network 192.168.40.0 255.255.255.0  
    R2(dhcp-config)# default-router 192.168.40.1  

    **Serial Interfaces configuration**  
    R2(config)# interface s0/1  
    R2(config-if)# ip address 172.16.0.2 255.255.255.0  
    R2(config-if)# no shutdown  

    R2(config)# interface s0/0  
    R2(config-if)# ip address 172.17.0.1 255.255.255.0  
    R2(config-if)# clock rate 64000  
    R2(config-if)# no shutdown  

    **Static Routing**  
    R2(config)# ip route 192.168.2.0 255.255.255.0 172.16.0.1    
    R2(config)# ip route 192.168.3.0 255.255.255.0 172.17.0.2  

    ## R3 Router Configuration

    Router> enable  
    Router# configure terminal  
    Router(config)# hostname R3  
    R3(config)# no ip domain-lookup  
    R3(config)# interface fa0/0  
    R3(config-if)# ip address 192.168.3.1 255.255.255.0  
    R3(config-if)# no shutdown  
    R3(config)# interface s0/1  
    R3(config-if)# ip address 172.17.0.2 255.255.255.0  
    R3(config-if)# no shutdown  

    **Static Routing**    
    R3(config)# ip route 192.168.2.0 255.255.255.0 172.17.0.1  

    ## Verification
    SW-50# show vlan brief → VLANs active

    SW-50# ping 192.168.20.50 → Management IP reachable

    R1# show ip route  
    R2# show ip route  
    R3# show ip route → Routing tables correct

    R1# show access-lists → ACL applied

    PC> ping <destination IP> → End‑to‑end connectivity

    **Note: Except R3 Lan can't communicate with R1 Lan due to Access-List**

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

