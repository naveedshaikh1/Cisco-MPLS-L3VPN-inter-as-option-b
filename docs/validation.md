# Verification commands and expected results

Run from privileged EXEC mode after all tasks. These are expected results, not fabricated captures. IOS command forms vary by release; use `?` or capture `show version` if rejected.

## 1. Basic configuration — all routers

```text
show version
show ip interface brief
show running-config
```

Required links must be up/up. CE LAN interfaces need an active attached link for their /24 routes to be present. If testing with hosts, choose e.g. 192.168.7.10/24 with gateway 192.168.7.1, and equivalently for the other LANs.

## 2. Provider OSPF and LDP — R1 through R6

```text
show ip ospf neighbor
show ip route ospf
show mpls ldp neighbor
show mpls interfaces
show mpls forwarding-table
```

| Router | Core OSPF neighbors | LDP neighbors |
| --- | --- | --- |
| R1 | R3 | R3 |
| R3 | R1, R4 | R1, R4 |
| R4 | R3 | R3 |
| R2 | R5 | R5 |
| R5 | R2, R6 | R2, R6 |
| R6 | R5 | R5 |

R4 and R6 also have a Customer B OSPF adjacency in process 100; the table lists only core process 1. Loopbacks must be passive and advertised. There should be no LDP neighbor across R1–R2.

R1:
```text
ping 1.1.0.4 source 1.1.0.1
```
R2:
```text
ping 1.1.0.6 source 1.1.0.2
```

## 3. BGP sessions — R1, R2, R4, R6

```text
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
show ip bgp vpnv4 all labels
```

| Router | Expected VPNv4 neighbors |
| --- | --- |
| R1 | 1.1.0.4 (AS 100), 12.12.12.2 (AS 200) |
| R2 | 1.1.0.6 (AS 200), 12.12.12.1 (AS 100) |
| R4 | 1.1.0.1 (AS 100) |
| R6 | 1.1.0.2 (AS 200) |

Established sessions normally display received-prefix counts rather than Idle/Active. Inspect RTs and labels in detailed route output. On R4, remote customer routes should resolve through 1.1.0.1; on R6, through 1.1.0.2.

R1 and R2:
```text
show running-config interface Ethernet0/0
show mpls interfaces
show mpls forwarding-table
```

Verify labeled border forwarding. Label numbers are dynamically assigned; do not expect specific values.

## 4. Customer VRFs and PE–CE routing

R4 and R6:
```text
show ip vrf interfaces
show ip route vrf Cust-A
show ip route vrf Cust-B
show ip eigrp vrf Cust-A neighbors
show ip ospf 100 neighbor
show ip bgp vpnv4 vrf Cust-A
show ip bgp vpnv4 vrf Cust-B
```

R7 and R9:
```text
show ip vrf interfaces
show ip eigrp vrf Cust-A neighbors
show ip route vrf Cust-A
```

R8 and R10:
```text
show ip vrf interfaces
show ip ospf 100 neighbor
show ip route vrf Cust-B
show running-config | section router ospf
```

Cust-A must contain remote LANs 192.168.7.0/24 and 192.168.9.0/24 as appropriate. Cust-B must contain 192.168.8.0/24 and 192.168.10.0/24. Local connected routes will not be displayed as learned routes. No unrelated customer LAN should appear in the wrong VRF.

## 5. Bidirectional reachability

On R7:
```text
ping vrf Cust-A 192.168.9.1 source 192.168.7.1
traceroute vrf Cust-A 192.168.9.1 source 192.168.7.1
```
On R9:
```text
ping vrf Cust-A 192.168.7.1 source 192.168.9.1
```
On R8:
```text
ping vrf Cust-B 192.168.10.1 source 192.168.8.1
traceroute vrf Cust-B 192.168.10.1 source 192.168.8.1
```
On R10:
```text
ping vrf Cust-B 192.168.8.1 source 192.168.10.1
```

Expect successful pings after convergence. Traceroute visibility depends on MPLS TTL behavior; every provider router need not appear. These pings test CE gateway reachability. Test actual attached hosts separately to validate host addressing, default gateways and firewalls.

## 6. Isolation checks

On R7:
```text
show ip route vrf Cust-A 192.168.10.1
ping vrf Cust-A 192.168.10.1 source 192.168.7.1
```
On R8:
```text
show ip route vrf Cust-B 192.168.9.1
ping vrf Cust-B 192.168.9.1 source 192.168.8.1
```

Expect no route to the other customer LAN and failed cross-customer pings. A failed ping alone does not prove isolation; inspect both PE VRF tables and import/export policies too.

## 7. Troubleshooting

| Symptom | Inspect |
| --- | --- |
| No OSPF neighbor | Interface status, address/mask, active interface, area and process |
| No LDP neighbor | MPLS enabled on core links, loopback reachability and LDP router ID |
| iBGP down | Loopback routes, update-source, AS number and VPNv4 activation |
| eBGP down | R1/R2 border IPs, AS numbers, VPNv4 activation |
| VPN routes absent on ASBR | Route-target filter setting, CE learning and PE redistribution |
| VPN routes present but missing in PE VRF | Import/export RT match |
| Route present but forwarding fails | ASBR next-hop-self, label forwarding, return path and MTU |
| EIGRP remote routes missing | EIGRP AS 10, VRF scope, BGP redistribution and metric |
| OSPF CE remote routes missing | VRF assignment, process 100, redistribution, CE-only capability vrf-lite |
| LAN prefix missing | CE LAN interface up/up, IP and routing network statement |

Record actual outputs in `evidence/`, including software version and test date. Save each working router with `copy running-config startup-config`.
