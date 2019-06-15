# Tasks and router commands — MPLS L3VPN Inter-AS Option B

## Scope and assumptions

These are newly reconstructed Cisco IOS-style lab configurations based on the supplied tasks and topology, not recovered historical exports. They have been checked for internal consistency but have not been executed on routers. Use a clean, isolated ten-router lab with an image supporting MPLS, VPNv4 BGP, VRF-aware OSPF and classic EIGRP address families. IOS XR syntax is different. Match interface names to your actual image before pasting.

Complete each task on every listed router before moving to the next task. Paste one router's block at a time from privileged EXEC mode. Alternatively, use the complete `configs/R1.cfg`–`R10.cfg` files, which combine the same task blocks. Do not mix these with an unknown existing router configuration.

## Corrections and implementation choices

| Item | Choice used here | Reason |
| --- | --- | --- |
| Second provider AS | 200 | Matches diagram and Task 3; “AS 102” in Task 2 is a typo |
| Option B border neighbors | R1 and R2 | They share 12.12.12.0/24; R4 and R6 provide the end-to-end customer VPN service |
| EIRGP | EIGRP | Spelling correction |
| EIGRP AS/process | 10 throughout Cust-A | Not specified; chosen for this reconstruction |
| Core OSPF | Process 1, area 0 | Consistent with diagram |
| Customer B OSPF | Process 100, area 0 | Required by tasks |
| LAN gateways | 192.168.7.1, .8.1, .9.1 and .10.1 in their respective /24s | Host addresses not specified; .1 selected per LAN |
| Customer router VRFs | VRF-lite on both CE uplink and LAN | Follows the explicit requirement to assign VRFs on R7/R8/R9/R10 too |
| Cust-A RD/import RT/export RT | 101:201 on R4 and R6 | Required by tasks |
| Cust-B RD/import RT/export RT | 102:202 on R4 and R6 | Required by tasks |

A conventional single-customer CE can instead use its global table. This implementation deliberately follows your CE VRF requirement; therefore CE verification must use `vrf Cust-A` or `vrf Cust-B`. CE VRFs use local RDs for consistency, but CEs do not run VPNv4 BGP, import/export RTs or MPLS.

## Task 1: Basic configuration

Configure hostnames, provider Loopback0 interfaces, all inter-router IP addresses and CE Ethernet LAN addresses. All physical links use /24 masks; provider loopbacks use /32 masks. Customer interfaces initially have global addresses; Task 3 moves them into VRFs and explicitly reapplies each address because `ip vrf forwarding` removes the existing IP address.

### R1

```text
enable
configure terminal
hostname R1
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.1 255.255.255.255
 exit
interface Ethernet0/0
 description R2 ASBR
 ip address 12.12.12.1 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R3 core
 ip address 1.1.13.1 255.255.255.0
 no shutdown
 exit
end
```

### R2

```text
enable
configure terminal
hostname R2
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.2 255.255.255.255
 exit
interface Ethernet0/0
 description R1 ASBR
 ip address 12.12.12.2 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R5 core
 ip address 2.2.25.2 255.255.255.0
 no shutdown
 exit
end
```

### R3

```text
enable
configure terminal
hostname R3
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.3 255.255.255.255
 exit
interface Ethernet0/0
 description R1 ASBR
 ip address 1.1.13.3 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R4 PE
 ip address 1.1.34.3 255.255.255.0
 no shutdown
 exit
end
```

### R4

```text
enable
configure terminal
hostname R4
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.4 255.255.255.255
 exit
interface Ethernet0/0
 description R3 core
 ip address 1.1.34.4 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R7 Cust-A
 ip address 192.168.47.4 255.255.255.0
 no shutdown
 exit
interface Ethernet0/2
 description R8 Cust-B
 ip address 172.16.48.4 255.255.255.0
 no shutdown
 exit
end
```

### R5

```text
enable
configure terminal
hostname R5
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.5 255.255.255.255
 exit
interface Ethernet0/0
 description R2 ASBR
 ip address 2.2.25.5 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R6 PE
 ip address 2.2.56.5 255.255.255.0
 no shutdown
 exit
end
```

### R6

```text
enable
configure terminal
hostname R6
no ip domain-lookup
ip cef
interface Loopback0
 ip address 1.1.0.6 255.255.255.255
 exit
interface Ethernet0/0
 description R5 core
 ip address 2.2.56.6 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description R10 Cust-B
 ip address 172.16.106.6 255.255.255.0
 no shutdown
 exit
interface Ethernet0/2
 description R9 Cust-A
 ip address 192.168.69.6 255.255.255.0
 no shutdown
 exit
end
```

### R7

```text
enable
configure terminal
hostname R7
no ip domain-lookup
ip cef
interface Ethernet0/0
 description R4 PE
 ip address 192.168.47.7 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description Customer A site 1 LAN
 ip address 192.168.7.1 255.255.255.0
 no shutdown
 exit
end
```

### R8

```text
enable
configure terminal
hostname R8
no ip domain-lookup
ip cef
interface Ethernet0/0
 description R4 PE
 ip address 172.16.48.8 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description Customer B site 1 LAN
 ip address 192.168.8.1 255.255.255.0
 no shutdown
 exit
end
```

### R9

```text
enable
configure terminal
hostname R9
no ip domain-lookup
ip cef
interface Ethernet0/0
 description R6 PE
 ip address 192.168.69.9 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description Customer A site 2 LAN
 ip address 192.168.9.1 255.255.255.0
 no shutdown
 exit
end
```

### R10

```text
enable
configure terminal
hostname R10
no ip domain-lookup
ip cef
interface Ethernet0/0
 description R6 PE
 ip address 172.16.106.10 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 description Customer B site 2 LAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
end
```
## Task 2: MPLS and IGP core routing

In AS 100, run OSPF area 0 and LDP on R1–R3 and R3–R4. In AS 200, run them on R2–R5 and R5–R6. Advertise all six provider loopbacks in OSPF as passive interfaces. `passive-interface default` keeps loopbacks passive, and only provider-facing links are made active.

LDP uses Loopback0 as its router ID. Do not enable OSPF or LDP on the R1–R2 inter-AS link; Task 4 uses BGP label signaling there. Do not enable MPLS on CE-facing interfaces.

### R1

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/1
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.1
 passive-interface default
 no passive-interface Ethernet0/1
 network 1.1.0.1 0.0.0.0 area 0
 network 1.1.13.1 0.0.0.0 area 0
 exit
end
```

### R2

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/1
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.2
 passive-interface default
 no passive-interface Ethernet0/1
 network 1.1.0.2 0.0.0.0 area 0
 network 2.2.25.2 0.0.0.0 area 0
 exit
end
```

### R3

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/0
 mpls ip
 exit
interface Ethernet0/1
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.3
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 network 1.1.0.3 0.0.0.0 area 0
 network 1.1.13.3 0.0.0.0 area 0
 network 1.1.34.3 0.0.0.0 area 0
 exit
end
```

### R4

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/0
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.4
 passive-interface default
 no passive-interface Ethernet0/0
 network 1.1.0.4 0.0.0.0 area 0
 network 1.1.34.4 0.0.0.0 area 0
 exit
end
```

### R5

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/0
 mpls ip
 exit
interface Ethernet0/1
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.5
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 network 1.1.0.5 0.0.0.0 area 0
 network 2.2.25.5 0.0.0.0 area 0
 network 2.2.56.5 0.0.0.0 area 0
 exit
end
```

### R6

```text
enable
configure terminal
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Ethernet0/0
 mpls ip
 exit
router ospf 1
 router-id 1.1.0.6
 passive-interface default
 no passive-interface Ethernet0/0
 network 1.1.0.6 0.0.0.0 area 0
 network 2.2.56.6 0.0.0.0 area 0
 exit
end
```
## Task 3: VPNv4 BGP, VRFs and PE–CE routing

Build loopback-sourced VPNv4 iBGP sessions R1↔R4 in AS 100 and R2↔R6 in AS 200. Define Cust-A and Cust-B on R4/R6 with the required RD/RT values, and assign the relevant interfaces. Define corresponding VRF-lite contexts on the CEs, including their LAN interfaces.

Run EIGRP AS 10 for Customer A and OSPF process 100 area 0 for Customer B. LAN interfaces are passive so they are advertised without trying to form customer LAN adjacencies. Redistribute between each PE's VRF IPv4 BGP address family and its customer routing process. VPNv4 carries the resulting VPN routes; the redistribution commands belong under the VRF IPv4 address families, not under `address-family vpnv4`.

The EIGRP seed metric is an explicit lab choice: bandwidth 100000 Kbit/s, delay 100 tens of microseconds, reliability 255, load 1, MTU 1500. It does not change interface MTU or bandwidth.

On VRF-lite CEs R8/R10, `capability vrf-lite` disables PE-specific OSPF checks that would otherwise reject some PE-advertised routes. Do not add it to the actual PEs R4/R6. No global `bgp redistribute-internal` is needed for this MPLS VPN VRF redistribution design.

### R1

```text
enable
configure terminal
router bgp 100
 bgp router-id 1.1.0.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 1.1.0.4 remote-as 100
 neighbor 1.1.0.4 update-source Loopback0
 address-family vpnv4
  neighbor 1.1.0.4 activate
  neighbor 1.1.0.4 send-community extended
 exit-address-family
 exit
end
```

### R2

```text
enable
configure terminal
router bgp 200
 bgp router-id 1.1.0.2
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 1.1.0.6 remote-as 200
 neighbor 1.1.0.6 update-source Loopback0
 address-family vpnv4
  neighbor 1.1.0.6 activate
  neighbor 1.1.0.6 send-community extended
 exit-address-family
 exit
end
```

### R4

```text
enable
configure terminal
router bgp 100
 bgp router-id 1.1.0.4
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 1.1.0.1 remote-as 100
 neighbor 1.1.0.1 update-source Loopback0
 address-family vpnv4
  neighbor 1.1.0.1 activate
  neighbor 1.1.0.1 send-community extended
 exit-address-family
 exit
ip vrf Cust-A
 rd 101:201
 route-target import 101:201
 route-target export 101:201
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-A
 ip address 192.168.47.4 255.255.255.0
 no shutdown
 exit
ip vrf Cust-B
 rd 102:202
 route-target import 102:202
 route-target export 102:202
 exit
interface Ethernet0/2
 ip vrf forwarding Cust-B
 ip address 172.16.48.4 255.255.255.0
 no shutdown
 exit
router eigrp 10
 address-family ipv4 vrf Cust-A
  autonomous-system 10
  eigrp router-id 1.1.0.4
  no auto-summary
  passive-interface default
  no passive-interface Ethernet0/1
  network 192.168.47.4 0.0.0.0
  redistribute bgp 100 metric 100000 100 255 1 1500
 exit-address-family
 exit
router ospf 100 vrf Cust-B
 router-id 1.1.0.4
 passive-interface default
 no passive-interface Ethernet0/2
 network 172.16.48.4 0.0.0.0 area 0
 redistribute bgp 100 subnets
 exit
router bgp 100
 address-family ipv4 vrf Cust-A
  redistribute eigrp 10
 exit-address-family
 address-family ipv4 vrf Cust-B
  redistribute ospf 100 match internal external 1 external 2
 exit-address-family
 exit
end
```

### R6

```text
enable
configure terminal
router bgp 200
 bgp router-id 1.1.0.6
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 1.1.0.2 remote-as 200
 neighbor 1.1.0.2 update-source Loopback0
 address-family vpnv4
  neighbor 1.1.0.2 activate
  neighbor 1.1.0.2 send-community extended
 exit-address-family
 exit
ip vrf Cust-A
 rd 101:201
 route-target import 101:201
 route-target export 101:201
 exit
interface Ethernet0/2
 ip vrf forwarding Cust-A
 ip address 192.168.69.6 255.255.255.0
 no shutdown
 exit
ip vrf Cust-B
 rd 102:202
 route-target import 102:202
 route-target export 102:202
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-B
 ip address 172.16.106.6 255.255.255.0
 no shutdown
 exit
router eigrp 10
 address-family ipv4 vrf Cust-A
  autonomous-system 10
  eigrp router-id 1.1.0.6
  no auto-summary
  passive-interface default
  no passive-interface Ethernet0/2
  network 192.168.69.6 0.0.0.0
  redistribute bgp 200 metric 100000 100 255 1 1500
 exit-address-family
 exit
router ospf 100 vrf Cust-B
 router-id 1.1.0.6
 passive-interface default
 no passive-interface Ethernet0/1
 network 172.16.106.6 0.0.0.0 area 0
 redistribute bgp 200 subnets
 exit
router bgp 200
 address-family ipv4 vrf Cust-A
  redistribute eigrp 10
 exit-address-family
 address-family ipv4 vrf Cust-B
  redistribute ospf 100 match internal external 1 external 2
 exit-address-family
 exit
end
```

### R7

```text
enable
configure terminal
ip vrf Cust-A
 rd 101:201
 exit
interface Ethernet0/0
 ip vrf forwarding Cust-A
 ip address 192.168.47.7 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-A
 ip address 192.168.7.1 255.255.255.0
 no shutdown
 exit
router eigrp 10
 address-family ipv4 vrf Cust-A
  autonomous-system 10
  eigrp router-id 192.168.7.1
  no auto-summary
  passive-interface default
  no passive-interface Ethernet0/0
  network 192.168.47.7 0.0.0.0
  network 192.168.7.1 0.0.0.0
 exit-address-family
 exit
end
```

### R8

```text
enable
configure terminal
ip vrf Cust-B
 rd 102:202
 exit
interface Ethernet0/0
 ip vrf forwarding Cust-B
 ip address 172.16.48.8 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-B
 ip address 192.168.8.1 255.255.255.0
 no shutdown
 exit
router ospf 100 vrf Cust-B
 router-id 192.168.8.1
 capability vrf-lite
 passive-interface default
 no passive-interface Ethernet0/0
 network 172.16.48.8 0.0.0.0 area 0
 network 192.168.8.1 0.0.0.0 area 0
 exit
end
```

### R9

```text
enable
configure terminal
ip vrf Cust-A
 rd 101:201
 exit
interface Ethernet0/0
 ip vrf forwarding Cust-A
 ip address 192.168.69.9 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-A
 ip address 192.168.9.1 255.255.255.0
 no shutdown
 exit
router eigrp 10
 address-family ipv4 vrf Cust-A
  autonomous-system 10
  eigrp router-id 192.168.9.1
  no auto-summary
  passive-interface default
  no passive-interface Ethernet0/0
  network 192.168.69.9 0.0.0.0
  network 192.168.9.1 0.0.0.0
 exit-address-family
 exit
end
```

### R10

```text
enable
configure terminal
ip vrf Cust-B
 rd 102:202
 exit
interface Ethernet0/0
 ip vrf forwarding Cust-B
 ip address 172.16.106.10 255.255.255.0
 no shutdown
 exit
interface Ethernet0/1
 ip vrf forwarding Cust-B
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
router ospf 100 vrf Cust-B
 router-id 192.168.10.1
 capability vrf-lite
 passive-interface default
 no passive-interface Ethernet0/0
 network 172.16.106.10 0.0.0.0 area 0
 network 192.168.10.1 0.0.0.0 area 0
 exit
end
```
## Task 4: Inter-AS Option B through R1 and R2

Connect the VPN service between R4 and R6 by exchanging labeled VPNv4 routes on the directly connected ASBRs R1 and R2.

- `no bgp default route-target filter` lets these ASBRs retain VPN routes without local customer VRFs.
- `send-community extended` carries the route-target information.
- `next-hop-self` toward the local PE makes the ASBR loopback the reachable next hop inside its own AS. This avoids importing the other provider's transport addressing into the local OSPF domain.
- `mpls bgp forwarding` permits labeled packets on the border interface. Some IOS releases install this automatically when VPNv4 eBGP comes up. Verify the command and forwarding state on your image; do not substitute `mpls ip` on this link just to clear a syntax error.

This deliberately broad route-retention setting is for the isolated lab. Production ASBRs require agreed VPN route policies and filtering.

### R1

```text
enable
configure terminal
interface Ethernet0/0
 mpls bgp forwarding
 exit
router bgp 100
 no bgp default route-target filter
 neighbor 12.12.12.2 remote-as 200
 address-family vpnv4
  neighbor 12.12.12.2 activate
  neighbor 12.12.12.2 send-community extended
  neighbor 1.1.0.4 next-hop-self
 exit-address-family
 exit
end
```

### R2

```text
enable
configure terminal
interface Ethernet0/0
 mpls bgp forwarding
 exit
router bgp 200
 no bgp default route-target filter
 neighbor 12.12.12.1 remote-as 100
 address-family vpnv4
  neighbor 12.12.12.1 activate
  neighbor 12.12.12.1 send-community extended
  neighbor 1.1.0.6 next-hop-self
 exit-address-family
 exit
end
```
## Save and verify

After checking for parser errors and completing the verification guide, save each router:

```text
copy running-config startup-config
```

See [verification commands](validation.md) for exact router checks, expected neighbor counts, positive pings and isolation tests. If a command is rejected, capture `show version`, the command and the full error before adapting the syntax. No actual test output is supplied in this package.

## References

- [Cisco: Inter-AS Option B configuration and verification](https://www.cisco.com/c/en/us/support/docs/multiprotocol-label-switching-mpls/mpls/200557-Configuration-and-Verification-of-Layer.html)
- [Cisco: Option B next-hop-self method](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9400/software/release/16-11/configuration_guide/mpls/b_1611_mpls_9400_cg/m9-1611-mpls-interas-option-b.html)
- [Cisco: EIGRP between PE and CE](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/mp_l3_vpns/configuration/xe-16-11/mp-l3-vpns-xe-16-11-book/mpls-vpn-support-for-eigrp-between-pe-and-ce.html)
- [Cisco: OSPF on VRF-lite CEs](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/15-mt/iro-15-mt-book/iro-sup-vrf.html)
