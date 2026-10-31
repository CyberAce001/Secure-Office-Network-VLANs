# Troubleshooting Log

Faults met while building and testing the network, with their cause and fix. Evidence screenshots are in [`screenshots/08_troubleshooting/`](../screenshots/08_troubleshooting/).

| # | Symptom | Cause | Resolution | Evidence |
|---|---|---|---|---|
| 1 | Router link red; Switch1 Fa0/5 `notconnect` although configured as a trunk | Router ports are shut down by default, so the far end of the link was down | `no shutdown` on Router0 Gi0/0; the link turned green | [`fault1_switch1_fa0-5_notconnect.png`](../screenshots/08_troubleshooting/fault1_switch1_fa0-5_notconnect.png), [`topology_router_link_down_before_config.png`](../screenshots/01_topology/topology_router_link_down_before_config.png) |
| 2 | `ip address` on G0/0.20 rejected with a message about 802.1Q | The address was entered before the subinterface had an encapsulation | Set `encapsulation dot1Q` first, then the address | [`fault2-3_subinterface_errors.png`](../screenshots/08_troubleshooting/fault2-3_subinterface_errors.png) |
| 3 | G0/0.20 given `encapsulation dot1Q 30` by mistake | Typing error | Re-entered `encapsulation dot1Q 20`, which replaces the old value; confirmed with `show interfaces` | [`fault2-3_subinterface_errors.png`](../screenshots/08_troubleshooting/fault2-3_subinterface_errors.png) |
| 4 | HR-PC1 could not ping FIN-PC1 (100% loss) | The gateway addresses did not exist yet because the subinterfaces were not finished | Completed Router-on-a-Stick; Test B then passed | [`fault4_inter_vlan_ping_before_routing.png`](../screenshots/08_troubleshooting/fault4_inter_vlan_ping_before_routing.png) |
| 5 | First ping of an inter-VLAN test times out | Expected: the first packet waits for ARP to resolve the gateway | None needed; the repeat ping succeeds | [`test_b_hr-pc1_to_it-pc1.png`](../screenshots/06_tests/test_b_hr-pc1_to_it-pc1.png) |
| 6 | An empty `router rip` process in the router configuration | Entered unintentionally; it had no `network` statements, so it had no effect | Removed with `no router rip`, saved, and the final configuration re-exported | [`fault6_unintended_router_rip.png`](../screenshots/08_troubleshooting/fault6_unintended_router_rip.png) |
| 7 | MAC address tables showed no PC entries | Dynamic entries had aged out after a period without traffic | Generated fresh pings and re-ran `show mac address-table` at once | [`fault7_switch1_mac_table_aged_out.png`](../screenshots/08_troubleshooting/fault7_switch1_mac_table_aged_out.png) |
| 8 | Simulation Mode reported *Buffer Full* | The event buffer filled during capture | Cleared the event list, reset the simulation, used a single Simple PDU stepped with Capture/Forward | [`fault8_simulation_buffer_full.png`](../screenshots/08_troubleshooting/fault8_simulation_buffer_full.png) |
