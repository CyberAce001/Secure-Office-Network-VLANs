# Screenshots

Evidence captured from Cisco Packet Tracer, reproduced unaltered (the black bar and red block on a few images are redactions made at capture time).

| Folder | Contents |
|---|---|
| `01_topology/` | Final topology (all links up) and the topology while the router link was still down |
| `02_vlans/` | `show vlan brief`, `show interfaces status` and `show mac address-table` for both switches |
| `03_trunks/` | `show interfaces trunk` for both switches |
| `04_hosts/` | IP configuration screens for the six PCs and Server0 |
| `05_router/` | `show ip interface brief`, `show ip route` and filtered `show interfaces` on Router0 |
| `06_tests/` | Tests A and B (pings), Test C (router link down and restored) and Test E (server reachability) |
| `07_simulation/` | Test D Simulation Mode PDU captures, numbered in the order of the packet journey |
| `08_troubleshooting/` | Evidence for the faults in the troubleshooting log |

## Simulation captures (Test D)

Two captures were taken. Run 1 started with empty ARP caches (ARP and ICMP). Run 2 was ICMP only (ARP cached), so its ICMP sequence numbers differ.

| File | What it shows |
|---|---|
| `01_arp_request_switch1_outbound` | ARP request for 192.168.10.1, broadcast destination, tagged VLAN 10 (Run 1) |
| `02_arp_reply_hr-pc1_inbound` | ARP reply carrying the router's MAC 0005.5EC8.2401 (Run 1) |
| `03_icmp_request_hr-pc1_outbound_run2` | Echo request addressed to the router's MAC (Run 2) |
| `04_icmp_request_router0_inbound` | 802.1Q frame tagged VLAN 10, TTL 255 (Run 1) |
| `05_icmp_request_router0_outbound` | MACs rewritten, tagged VLAN 30, TTL 254 (Run 1) |
| `06_icmp_request_switch1_outbound_untagged` | Untagged Ethernet II frame toward FIN-PC1 (Run 1) |
| `07_icmp_request_fin-pc1_inbound` | Echo request arriving, IP addresses unchanged (Run 1) |
| `08_icmp_reply_fin-pc1_outbound` | Echo reply, 192.168.30.11 to 192.168.10.11 (Run 1) |
| `09_icmp_reply_hr-pc1_inbound_run2` | Echo reply from the router's MAC, TTL 127 (Run 2) |
| `10_icmp_request_switch1_inbound_vlan30_tag` | Frame returning from the router with the VLAN 30 tag (Run 1, extra) |
| `11_arp_request_fin-pc1_outbound` | FIN-PC1 resolving its own gateway 192.168.30.1 before replying (Run 1, extra) |
