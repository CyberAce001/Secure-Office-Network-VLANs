# Implementation Guide

Step-by-step build of the network, written so it can be repeated from scratch in Cisco Packet Tracer. Each step ends with a check. Commands assume the default hostnames (`Switch` for each switch, `Router` for the router); the prompts are shortened accordingly.

**Addressing reference**

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | HR | 192.168.10.0/24 | 192.168.10.1 (G0/0.10) |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 (G0/0.20) |
| 30 | FINANCE | 192.168.30.0/24 | 192.168.30.1 (G0/0.30) |
| none | Servers | 192.168.40.0/24 | 192.168.40.1 (G0/1) |

| Device | Connected to | VLAN | IP address |
|---|---|---|---|
| HR-PC1 | Switch1 Fa0/1 | 10 | 192.168.10.11 |
| IT-PC1 | Switch1 Fa0/2 | 20 | 192.168.20.11 |
| FIN-PC1 | Switch1 Fa0/3 | 30 | 192.168.30.11 |
| HR-PC2 | Switch2 Fa0/1 | 10 | 192.168.10.12 |
| IT-PC2 | Switch2 Fa0/2 | 20 | 192.168.20.12 |
| FIN-PC2 | Switch2 Fa0/3 | 30 | 192.168.30.12 |
| Server0 | Router0 Gi0/1 | none | 192.168.40.10 |

All masks are 255.255.255.0.

## Step 1: Build the topology

- Place one Cisco 2911 router, two Cisco 2960-24TT switches, six PCs and one server.
- Name the PCs HR-PC1, IT-PC1, FIN-PC1, HR-PC2, IT-PC2, FIN-PC2 and the switches Switch1 and Switch2.
- Cable them as follows. Router interfaces are shut down by default, so the router links stay red until Step 5.

| Link | Endpoint A | Endpoint B | Cable |
|---|---|---|---|
| Switch to switch | Switch1 Fa0/4 | Switch2 Fa0/4 | Cross-over |
| Router uplink | Switch1 Fa0/5 | Router0 Gi0/0 | Straight-through |
| Server link | Router0 Gi0/1 | Server0 Fa0 | Cross-over |
| PC links | Fa0/1 to Fa0/3 on each switch | PC FastEthernet0 | Straight-through |

## Step 2: Create the VLANs and assign access ports

Run on **both** switches. The same commands are in [`configs/vlan_database.txt`](../configs/vlan_database.txt).

```
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name IT
Switch(config-vlan)# exit
Switch(config)# vlan 30
Switch(config-vlan)# name FINANCE
Switch(config-vlan)# exit
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit
Switch(config)# interface fa0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 30
Switch(config-if)# end
```

**Check:** `show vlan brief` lists VLANs 10, 20 and 30 as active, with Fa0/1, Fa0/2 and Fa0/3 under them.

> VLAN definitions are stored in the VLAN database, not in the running configuration. That is why the exported configuration files do not contain the `vlan` lines, and why this step must be repeated on a rebuilt switch.

## Step 3: Configure the trunks

**Switch1**

```
Switch(config)# interface fa0/4
Switch(config-if)# description Trunk to Switch2
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# exit
Switch(config)# interface fa0/5
Switch(config-if)# description Trunk to Router G0/0
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# end
```

**Switch2**

```
Switch(config)# interface fa0/4
Switch(config-if)# description Trunk to Switch 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# end
```

A 2960 in Packet Tracer supports only 802.1Q, so no encapsulation command is needed (entering one gives an error). The allowed-VLAN list is optional, but it keeps unused VLANs off the trunks.

**Check:** `show interfaces trunk` shows each trunk port as `trunking`, with VLANs 10, 20 and 30 allowed.

## Step 4: Address the hosts

On each PC open **Desktop > IP Configuration**, choose **Static**, and enter the address, mask and default gateway from the table above. Do the same on Server0 with 192.168.40.10, mask 255.255.255.0 and gateway 192.168.40.1.

**Check:** ping from HR-PC1 to HR-PC2 (192.168.10.12). It succeeds, because both are in VLAN 10 and the trunk joins the switches. Pinging a gateway does not work yet.

## Step 5: Configure Router-on-a-Stick

```
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# no shutdown
Router(config-if)# exit
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface g0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
Router(config-subif)# end
```

- **Enter `encapsulation` before the IP address.** The router rejects an address on a subinterface that has no VLAN encapsulation yet.
- **Leave the physical interface without an IP address.** Only the subinterfaces carry addresses.

**Check:** `show ip interface brief` shows G0/0.10, G0/0.20 and G0/0.30 with their gateway addresses as `up/up`, and the router links turn green. Pings between VLANs now succeed.

## Step 6: Configure the server network

```
Router(config)# interface g0/1
Router(config-if)# ip address 192.168.40.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# end
```

The server uses a routed port, not a VLAN, so its traffic is untagged. The subnet 192.168.40.0/24 was chosen to follow the existing pattern.

## Step 7: Save

```
Switch# write memory
Router# write memory
```

Run on all three devices, then save the Packet Tracer file. Without this, the configuration is lost if a device restarts.

## Verification commands

| Device | Command | What it confirms |
|---|---|---|
| Switches | `show vlan brief` | VLANs exist, are active, and have the right access ports |
| Switches | `show interfaces trunk` | Trunk ports, encapsulation, native VLAN, allowed VLANs |
| Switches | `show interfaces status` | Link state and whether each port is access or trunk |
| Switches | `show mac address-table` | MAC addresses learned per VLAN and port |
| Router | `show ip interface brief` | Interface addresses and up/up status |
| Router | `show ip route` | Connected routes for every subnet |
| Router | `show interfaces g0/0.10` | MAC address, IP address and 802.1Q VLAN ID |
| All | `show running-config` | The complete active configuration |

Tip for the router: `show interfaces g0/0.10 | include line protocol|address|Encapsulation` trims the output to the lines that matter.
