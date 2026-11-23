# Connectivity Test Report — Pre-ACL Baseline

**Test stage:** Before access-control changes  
**Environment:** Cisco Packet Tracer  
**Evidence status:** Success was reported in the conversation; complete ping console screenshots were not provided.

| ID | Source | Destination | Result | Evidence |
| --- | --- | --- | --- | --- |
| T01 | Router1 | 192.168.100.2 | PASS (reported) | User confirmed success |
| T02 | PC1 | 172.16.10.254 | PASS (reported) | User confirmed success |
| T03 | PC1 | 172.16.10.1 (PC3) | PASS (reported) | User confirmed success |
| T04 | VLAN 10 host | VLAN 20 host | NOT TESTED | Required before ACL deployment |

## Reproduction checklist

- [ ] Save actual terminal screenshot of T01
- [ ] Save actual terminal screenshot of T02
- [ ] Save actual terminal screenshot of T03
- [ ] Verify host addressing and default gateways
- [ ] Confirm VLAN 10 → VLAN 20 baseline behavior
- [ ] After ACL deployment, repeat and record all relevant tests

**Interpretation:** Passing T03 indicates PC1-to-PC3 ICMP reachability at test time. It does not prove unrestricted connectivity to every network or service.
