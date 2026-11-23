# LAB 03 — Cisco IOS Configuration Notes

These are the commands used in the lab walkthrough. They are not a verified full running-configuration dump.

## Router1 — DHCP

```cisco
configure terminal
ip dhcp excluded-address 10.10.10.254
ip dhcp excluded-address 192.168.1.254
ip dhcp pool VLAN10_POOL
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.254
 dns-server 8.8.8.8
exit
ip dhcp pool VLAN20_POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.254
 dns-server 8.8.8.8
exit
end
```

## Router1 — ISP interface, NAT/PAT and default route

```cisco
configure terminal
interface GigabitEthernet0/2
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit
access-list 10 permit 10.10.10.0 0.0.0.255
access-list 10 permit 192.168.1.0 0.0.0.255
interface GigabitEthernet0/0.10
 ip nat inside
exit
interface GigabitEthernet0/0.20
 ip nat inside
exit
interface GigabitEthernet0/2
 ip nat outside
exit
ip nat inside source list 10 interface GigabitEthernet0/2 overload
ip route 0.0.0.0 0.0.0.0 203.0.113.1
end
```

## ISP-Router — Simulated ISP links

```cisco
configure terminal
interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 ip address 198.51.100.1 255.255.255.0
 no shutdown
exit
end
```

## Internet-Server — Static network settings

```text
IPv4 Address: 198.51.100.10
Subnet Mask: 255.255.255.0
Default Gateway: 198.51.100.1
```

## Verification commands

```cisco
show ip dhcp binding
show ip interface brief
show ip nat statistics
show ip nat translations
```

Note: `show ip nat statistics` was captured before traffic and therefore initially reported zero translations. The later translation screenshots show successful ICMP NAT entries.
