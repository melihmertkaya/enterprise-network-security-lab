# LAB 03 — Enterprise DHCP, NAT/PAT & Simulated Internet Access

## Objective

Extend the existing VLAN-based enterprise topology to assign client network settings automatically via Cisco IOS DHCP, connect Router1 to a simulated ISP, and use Port Address Translation (PAT) to allow multiple internal VLANs to reach a test server through one outside IP.

## Topology

```text
PC1 (VLAN 10, DHCP) ----\
                           Router1 (NAT/PAT) -- ISP-Router -- Internet-Server
PC2 (VLAN 20, DHCP) ----/       |              203.0.113.1      198.51.100.10
                                |
                           Router2 (existing LAB 01/02 path; retained)
```

Router1's original Router2 link and the LAB 02 inbound extended ACL were retained. The simulated ISP uses a separate Router1 interface.

## IP addressing plan

| Device / segment | Interface / role | Address / subnet |
| --- | --- | --- |
| PC1 / VLAN 10 | DHCP client | `10.10.10.3/24` (observed lease) |
| Router1 | `Gi0/0.10` gateway | `10.10.10.254/24` |
| PC2 / VLAN 20 | DHCP client | `192.168.1.1/24` (observed lease) |
| Router1 | `Gi0/0.20` gateway | `192.168.1.254/24` |
| Router1 ↔ Router2 | `Gi0/1` | `192.168.100.1/30` on Router1 |
| Router1 ↔ ISP | `Gi0/2` outside | `203.0.113.2/30` |
| ISP-Router ↔ Router1 | `Gi0/0` | `203.0.113.1/30` |
| ISP-Router ↔ server | `Gi0/1` | `198.51.100.1/24` |
| Internet-Server | `FastEthernet0` | `198.51.100.10/24`; gateway `198.51.100.1` |

**Important:** `203.0.113.0/24` and `198.51.100.0/24` are RFC 5737 documentation networks used to simulate public addressing. No real internet access was established. The DHCP clients were configured to receive DNS `8.8.8.8`, but DNS name resolution was **not** tested.

## Configuration

See [Router1 and ISP-Router command reference](configurations/CONFIGURATION-NOTES.md). This records commands applied during the guided lab, not a complete `show running-config` export.

## Validation evidence

| Check | Observed outcome | Evidence |
| --- | --- | --- |
| PC1 DHCP lease | Router1 binding: `10.10.10.3`, Automatic | [DHCP bindings and interfaces](evidence/01-dhcp-bindings-interfaces.png) |
| PC2 DHCP lease | Router1 binding: `192.168.1.1`, Automatic | [DHCP bindings and interfaces](evidence/01-dhcp-bindings-interfaces.png) |
| Router1 ↔ ISP-Router | Ping reported successful | Reported during lab; no retained ping screenshot |
| ISP-Router ↔ Internet-Server | Ping reported successful | Reported during lab; no retained ping screenshot |
| PC2 PAT | `192.168.1.1` translated to `203.0.113.2` for ICMP traffic to `198.51.100.10` | [PC2 NAT translations](evidence/02-pc2-nat-translations.png) |
| PC1 PAT | `10.10.10.3` translated to `203.0.113.2` for ICMP traffic to `198.51.100.10` | [PC1 NAT translations](evidence/03-pc1-nat-translations.png) |

The NAT screenshots prove translation entries for each client, captured in separate tests. They do **not** prove both clients were simultaneously active.

## Troubleshooting note

Router1 displayed `%DHCPD-4-PING_CONFLICT: DHCP address conflict: server pinged 10.10.10.2.` This means the DHCP server detected a possible existing use of `10.10.10.2` while checking an address. The subsequent lease table showed working assignments for `10.10.10.3` and `192.168.1.1`. The root cause of the conflict was **not investigated or resolved** in this lab.

## Key concepts demonstrated

- Cisco IOS DHCP pools and excluded gateway addresses
- DHCP assignment across two router subinterfaces / VLANs
- NAT inside and outside interface roles
- Standard ACL used to select NAT-eligible source subnets (distinct from the security ACL in LAB 02)
- PAT (`overload`) using Router1's `Gi0/2` outside address
- Default route toward the simulated ISP
- Verification with `show ip dhcp binding`, `show ip nat statistics`, and `show ip nat translations`

## Project file

Add the saved Packet Tracer file **`lab-03-dhcp-nat.pkt`** to this folder. It is not bundled here because the local file has not been uploaded to this conversation.
