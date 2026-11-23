# Lab 02 — Extended ACL & Network Segmentation

## Overview

This lab extends **Lab 01 — Enterprise Network Design & Routing** by introducing an extended IPv4 access control list (ACL) on Router1. The goal is to restrict traffic originating from VLAN 10 toward VLAN 20 while preserving connectivity to a remote subnet through Router2.

**Status:** ACL configuration and the specified connectivity outcomes were reported as successful. The attached screenshot independently confirms the ACL is applied inbound on Router1's `GigabitEthernet0/0.10` subinterface. Packet-level captures and ping screenshots have not yet been added.

## Network context

| Component | Address / network | Role |
|---|---|---|
| VLAN 10 | `10.10.10.0/24` | Source segment |
| PC1 | `10.10.10.1/24` | Test source in VLAN 10 |
| Router1 G0/0.10 | `10.10.10.254/24` | VLAN 10 gateway; ACL ingress |
| VLAN 20 | `192.168.1.0/24` | Restricted destination segment |
| PC2 | `192.168.1.1` | Test destination in VLAN 20 |
| Router1 G0/0.20 | `192.168.1.254/24` | VLAN 20 gateway |
| Router1–Router2 | `192.168.100.0/30` | Inter-router transit network |
| Remote network | `172.16.10.0/24` | Allowed destination segment |
| PC3 | `172.16.10.1` | Remote connectivity test destination |

The topology and baseline routing design are documented in Lab 01.

## Security requirement

- **Deny** IPv4 traffic initiated from `10.10.10.0/24` toward `192.168.1.0/24` when entering Router1 through the VLAN 10 subinterface.
- **Permit** other IPv4 traffic through this ACL, including PC1-to-PC3 communication.
- Preserve the routing established in Lab 01.

> **Scope:** This is a directional, stateless ACL policy applied on VLAN 10 ingress. It does not constitute a bidirectional, stateful firewall policy or a blanket isolation guarantee for all possible traffic paths.

## Configuration — Router1

The following is the ACL configuration used during the exercise:

```cisco
configure terminal
ip access-list extended BLOCK_VLAN10_TO_VLAN20
 deny ip 10.10.10.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip any any
exit
interface GigabitEthernet0/0.10
 ip access-group BLOCK_VLAN10_TO_VLAN20 in
exit
end
```

### Why these commands?

- `ip access-list extended ...` creates a named extended IPv4 ACL.
- `deny ip ... ...` matches traffic by both source and destination network. The wildcard `0.0.0.255` corresponds to a `/24` network.
- `permit ip any any` permits other IPv4 traffic; without an explicit permit, the ACL's implicit deny would block remaining unmatched traffic.
- `ip access-group ... in` evaluates packets arriving from VLAN 10 as they enter Router1, before they are routed toward their destination.

## Verification

### Interface-level evidence

The screenshot [`evidence/01-acl-applied-interface.png`](evidence/01-acl-applied-interface.png) shows the output of:

```cisco
show ip interface GigabitEthernet0/0.10
```

Observed values:

```text
GigabitEthernet0/0.10 is up, line protocol is up (connected)
Internet address is 10.10.10.254/24
Outgoing access list is not set
Inbound access list is BLOCK_VLAN10_TO_VLAN20
```

This confirms **placement and direction** of the ACL. It does not, by itself, prove the ACL rule contents or packet matches.

### Connectivity tests

| Test | Before ACL | After ACL | Evidence status |
|---|---|---|---|
| PC1 (`10.10.10.1`) → PC2 (`192.168.1.1`) | Ping succeeded | Ping failed; Router1 (`10.10.10.254`) returned `Destination host unreachable` | User-reported test and pasted output; screenshot pending |
| PC1 (`10.10.10.1`) → PC3 (`172.16.10.1`) | Ping succeeded | Ping succeeded | User-reported; screenshot pending |

These results are consistent with the intended ACL policy. A `Destination host unreachable` response alone does not uniquely prove an ACL drop; the comparison with the previously successful baseline and the applied ACL provide supporting context.

### Suggested follow-up commands

```cisco
show ip access-lists
show running-config interface GigabitEthernet0/0.10
```

If supported by the Packet Tracer device, record rule match counters after running both ping tests. Counter availability and behavior can vary by IOS simulation.

## Rollback

To remove the ACL from the interface without deleting its definition:

```cisco
configure terminal
interface GigabitEthernet0/0.10
 no ip access-group BLOCK_VLAN10_TO_VLAN20 in
end
```

Verify restored connectivity from PC1 to PC2. Reapply the ACL if continuing the lab.

## Evidence checklist

- [x] Screenshot: Router1 VLAN 10 subinterface with inbound ACL
- [x] PC1 → PC2 before/after results recorded in the lab discussion
- [x] PC1 → PC3 before/after results recorded in the lab discussion
- [ ] Screenshot: `show ip access-lists` showing ACL rules (and match counters if available)
- [ ] Screenshot: PC1 → PC2 blocked ping
- [ ] Screenshot: PC1 → PC3 permitted ping
- [ ] Save and add **ACL-enabled** Packet Tracer project as `lab-02-extended-acl.pkt`

## Lessons learned

1. Extended ACLs can filter traffic using source and destination addresses.
2. Applying an ACL inbound near the source makes the intended filtering point explicit.
3. A security change should be validated against both a **blocked** path and an **allowed** path.
4. Baseline test results and screenshots make troubleshooting and technical reporting more credible.

## Next step

Capture the remaining CLI and ping evidence, then save the modified Packet Tracer topology under a new filename. **Do not reuse the Lab 01 `.pkt` file as evidence of ACL configuration without verifying that the ACL is saved in it.**
