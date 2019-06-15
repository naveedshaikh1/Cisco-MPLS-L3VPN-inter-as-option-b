# Addressing inventory

Transcribed from the original topology. Host addresses below combine the subnet labels with the endpoint suffixes shown in the image; verify them against the original running configurations before use.

| Endpoint 1 | Address | Endpoint 2 | Address | Subnet |
| --- | --- | --- | --- | --- |
| R1 e0/0 | 12.12.12.1 | R2 e0/0 | 12.12.12.2 | 12.12.12.0/24 |
| R1 e0/1 | 1.1.13.1 | R3 e0/0 | 1.1.13.3 | 1.1.13.0/24 |
| R3 e0/1 | 1.1.34.3 | R4 e0/0 | 1.1.34.4 | 1.1.34.0/24 |
| R2 e0/1 | 2.2.25.2 | R5 e0/0 | 2.2.25.5 | 2.2.25.0/24 |
| R5 e0/1 | 2.2.56.5 | R6 e0/0 | 2.2.56.6 | 2.2.56.0/24 |
| R4 e0/1 | 192.168.47.4 | R7 e0/0 | 192.168.47.7 | 192.168.47.0/24 |
| R4 e0/2 | 172.16.48.4 | R8 e0/0 | 172.16.48.8 | 172.16.48.0/24 |
| R6 e0/1 | 172.16.106.6 | R10 e0/0 | 172.16.106.10 | 172.16.106.0/24 |
| R6 e0/2 | 192.168.69.6 | R9 e0/0 | 192.168.69.9 | 192.168.69.0/24 |

## Customer LANs

| Customer | Site | CE LAN interface | Subnet | PE–CE protocol |
| --- | --- | --- | --- | --- |
| A | 1 | R7 e0/1 | 192.168.7.0/24 | EIGRP |
| A | 2 | R9 e0/1 | 192.168.9.0/24 | EIGRP |
| B | 1 | R8 e0/1 | 192.168.8.0/24 | OSPF |
| B | 2 | R10 e0/1 | 192.168.10.0/24 | OSPF |

The user-supplied tasks specify Cust-A (RD/RT 101:201), Cust-B (RD/RT 102:202), and customer OSPF process 100 area 0. For the reconstructed commands, EIGRP AS 10 is selected and each CE LAN gateway uses .1 (192.168.7.1, 192.168.8.1, 192.168.9.1 and 192.168.10.1). These chosen values are not claimed to be historical values. See tasks-and-commands.md for all assumptions.
