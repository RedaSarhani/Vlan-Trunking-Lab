# VLAN Trunking Lab — Cisco Packet Tracer

## Objective
Build a multi-switch topology with 3 VLANs + default VLAN, using
trunk links between switches and access links to end devices, then
verify inter-VLAN and cross-switch connectivity.

## Topology
![topology](topology.png)

- **Switch0** (distribution/server switch): VLAN 10, VLAN 30
- **Switch1**: connects Switch0, Switch2, and Router0
- **Switch2**: VLAN 20
- Router0: connected to Switch1 via Gig0/0 for inter-VLAN routing

## Configuration
- Switch-to-switch links: trunk mode (802.1Q)
- Switch-to-PC links: access mode, assigned per VLAN
- Subnets: 192.168.1.0 (Switch0 side), 192.168.2.0 (Switch1 side), 192.168.3.0 (Switch2 side)

## What I tested
| Source | Destination | VLAN    | Result     |
|--------|-------------|---------|------------|
| PC0    | PC0(2)      | 10 → 30 | Success    |
| PC0(2) | PC0(3)      | 30      | **Failed** |

## Known issue / what I haven't solved yet
PC0(2) and PC0(3) are both on VLAN 30 but sit in different IP subnets
and connect through different switches. Ping fails between them.
My working theory: [write your current best guess here — e.g. "VLAN 30
may not be carried across the Switch0–Switch1 trunk" or "no SVI/router
interface exists for VLAN 30's subnet on Router0"].

## Next steps
- Verify VLAN 30 is allowed on the trunk (`show interfaces trunk`)
- Confirm each switch has VLAN 30 in its VLAN database
- Check Router0 sub-interfaces / SVI configuration for VLAN 30's subnet

## Defender's view
From a monitoring standpoint, this lab shows why VLAN segmentation
alone isn't enough — trunk misconfigurations or missing inter-VLAN
routes are common misconfig findings during network audits, and
would show up as unexpected connectivity failures or, worse,
unintended VLAN leakage if trunks are misconfigured to allow too much.
