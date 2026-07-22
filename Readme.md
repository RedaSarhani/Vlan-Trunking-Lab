# VLAN Trunking Lab — Cisco Packet Tracer

## Objective
Build a multi-switch topology with 3 VLANs + default VLAN, using
trunk links between switches and access links to end devices, then
verify inter-VLAN and cross-switch connectivity.

## Topology
<img width="1917" height="865" alt="image" src="https://github.com/user-attachments/assets/eac32898-e7bd-441f-b55b-6e63a82fec95" />

## Devices used
- 1 Router (Router0)
- 3 Switches (Switch0, Switch1, Switch2)
- 7 PCs

## Step 1: Building the topology
I placed the devices and connected them like this:
- PC0 and PC0(1) → connected to Switch0
- PC0(2) → connected to Switch0
- PC0(3) and PC0(4) → connected to Switch1
- PC0(5) and PC0(6) → connected to Switch2
- **Switch0** (distribution/server switch): VLAN 10, VLAN 30
- **Switch1**: connects Switch0, Switch2, and Router0
- **Switch2**: VLAN 20
- Router0: connected to Switch1 via Gig0/0 for inter-VLAN routing

## Configuration
- Switch-to-switch links: trunk mode (802.1Q)
- Switch-to-PC links: access mode, assigned per VLAN
- Subnets: 192.168.1.0 (Switch0 side), 192.168.2.0 (Switch1 side), 192.168.3.0 (Switch2 side)

<img width="885" height="837" alt="image" src="https://github.com/user-attachments/assets/f7171628-6a96-4557-b946-e1b83f5e16e7" />

## Step 2: Creating the VLANs
On each switch, I created the VLANs using this command:
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name VLAN10
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name VLAN20
Switch(config-vlan)# exit
Switch(config)# vlan 30
Switch(config-vlan)# name VLAN30
Switch(config-vlan)# exit

I repeated this on every switch so all of them know about VLAN 10, 20, and 30.
<img width="851" height="827" alt="image" src="https://github.com/user-attachments/assets/0c18236d-27a5-40bc-b347-8e910e18eb14" />
<img width="776" height="460" alt="image" src="https://github.com/user-attachments/assets/b7d0e831-9853-41f5-a2b2-a75cddcfb887" />

## Step 3: Configuring VTP (server/client mode)
I set Switch0 as the VTP server and the others as VTP clients, so 
VLANs only need to be created once and get shared automatically:

! On Switch0 (server)
Switch(config)# vtp mode server
Switch(config)# vtp domain gomycode
Switch(config)# vtp password 123456

! On Switch1 and Switch2 (clients)
Switch(config)# vtp mode client
Switch(config)# vtp domain gomycode
Switch(config)# vtp password 123456

<img width="726" height="297" alt="image" src="https://github.com/user-attachments/assets/e2be5ac0-65ab-4066-8633-9d4745c759e7" />

## Step 4: Assigning access ports (switch to PC)
For each port connected to a PC, I set it to access mode and put it 
in the right VLAN:
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

I did this for every PC port, matching the VLAN it belongs to:
- Switch0: ports to PC0 and PC0(1) → VLAN 10, port to PC0(2) → VLAN 30
- Switch1: port to PC0(3) → VLAN 30, port to PC0(4) → default VLAN
- Switch2: ports to PC0(5) and PC0(6) → VLAN 20

## Step 5: Configuring trunk ports (switch to switch)
For the links between switches, I set the ports to trunk mode so 
they can carry traffic for all VLANs:
Switch(config)# interface fastEthernet 0/3
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

<img width="650" height="197" alt="image" src="https://github.com/user-attachments/assets/c57dbfe3-b904-4f64-8d39-4d1641cd0ed1" />

## What I tested
| Source | Destination | VLAN    | Result     |
|--------|-------------|---------|------------|
| PC0    | PC0(2)      | 10 → 30 | Success    |
| PC0(2) | PC0(3)      | 30      | **Failed** |

<img width="617" height="122" alt="image" src="https://github.com/user-attachments/assets/162e790b-9ccd-40bf-abd7-b53555837800" />


## Known issue / what I haven't solved yet
PC0(2) and PC0(3) are both on VLAN 30 but sit in different IP subnets
and connect through different switches. Ping fails between them.
My working theory: one VLAN should map to one subnet only.
In my setup, VLAN 30 is split across two different subnets on two
different switches, which is not standard practice and is likely the
real cause of the problem, separate from any trunk or routing config.

I think the fix is either:
1. Redesign the lab so VLAN 30 uses one single subnet across both
   switches,
2. Accept that this is inter-subnet traffic and configure Router0
   properly to route between 192.168.1.0 and 192.168.2.0

## Defender's view
From a monitoring standpoint, this lab shows why VLAN segmentation
alone isn't enough — trunk misconfigurations or missing inter-VLAN
routes are common misconfig findings during network audits, and
would show up as unexpected connectivity failures or, worse,
unintended VLAN leakage if trunks are misconfigured to allow too much.

## Tools used
Cisco Packet Tracer
