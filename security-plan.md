# ACL Change Plan — Pending Approval and Implementation

## Business requirement
Prevent VLAN 10 clients (`10.10.10.0/24`) from initiating IP traffic to VLAN 20 (`192.168.1.0/24`) while preserving other permitted paths, including the remote network `172.16.10.0/24`.

## Proposed Cisco IOS configuration (NOT YET APPLIED)

```cisco
ip access-list extended BLOCK_VLAN10_TO_VLAN20
 deny ip 10.10.10.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip any any
exit
interface GigabitEthernet0/0.10
 ip access-group BLOCK_VLAN10_TO_VLAN20 in
```

## Pre-change requirements

- Verify actual VLAN membership and trunk tagging on switches.
- Record pre-change VLAN 10 → VLAN 20 and VLAN 10 → remote network tests.
- Back up the `.pkt` project.
- Confirm ACL name is not already used and interface configuration is as expected.

## Post-change validation

1. `show ip access-lists`
2. `show ip interface GigabitEthernet0/0.10`
3. From a confirmed VLAN 10 host, test a confirmed VLAN 20 host (expected blocked).
4. From the same VLAN 10 host, test PC3 `172.16.10.1` (expected allowed).
5. Check counters and document actual results. A blocked ping alone does not prove the ACL was responsible.

## Rollback

```cisco
interface GigabitEthernet0/0.10
 no ip access-group BLOCK_VLAN10_TO_VLAN20 in
```

**Scope caveat:** This inbound ACL on VLAN 10 is directional. It is not a complete bidirectional segmentation policy, and `permit ip any any` is a lab-specific rule rather than a least-privilege production policy.
