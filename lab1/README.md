# Распределение адресного пронстранства 
## Config

| SPINE1 | SPINE2 | LEAF1 | LEAF2 | LEAF3 | |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `enable` | `enable` | `enable` | `enable` | `enable` | **Host1:** |
| `configure terminal` | `configure terminal` | `configure terminal` | `configure terminal` | `configure terminal` | IP: 192.168.1.10/24 |
| | | | | | GW: 192.168.1.1 |
| `hostname Spine1` | `hostname Spine2` | `hostname Leaf1` | `hostname Leaf2` | `hostname Leaf3` | **Host2:** |
| `ip routing` | `ip routing` | `ip routing` | `ip routing` | `ip routing` | IP: 192.168.2.10/24 |
| | | | | | GW: 192.168.2.1 |
| `interface Ethernet1` | `interface Ethernet1` | `interface Ethernet1` | `interface Ethernet1` | `interface Ethernet1` | **Host3:** |
| `description TO-Leaf1-Eth1` | `description TO-Leaf1-Eth2` | `description TO-Spine1-Eth1` | `description TO-Spine1-Eth2` | `description TO-Spine1-Eth3` | IP: 192.168.3.10/24 |
| `no switchport` | `no switchport` | `no switchport` | `no switchport` | `no switchport` | GW: 192.168.3.1 |
| `ip address` | `ip address` | `ip address` | `ip address` | `ip address` | |
| `no shutdown` | `no shutdown` | `no shutdown` | `no shutdown` | `no shutdown` | **Host4:** |
| | | | | | IP: 192.168.3.11/24 |
| `interface Ethernet2` | `interface Ethernet2` | `interface Ethernet2` | `interface Ethernet2` | `interface Ethernet2` | GW: 192.168.3.1 |
| `description TO-Leaf2-Eth1` | `description TO-Leaf2-Eth2` | `description TO-Spine2-Eth1` | `description TO-Spine2-Eth2` | `description TO-Spine2-Eth3` | |
| `no switchport` | `no switchport` | `no switchport` | `no switchport` | `no switchport` | |
| `ip address` | `ip address` | `ip address` | `ip address` | `ip address` | |
| `no shutdown` | `no shutdown` | `no shutdown` | `no shutdown` | `no shutdown` | |
| | | | | | |
| `interface Ethernet3` | `interface Ethernet3` | `vlan 10` | `vlan 20` | `vlan 30` | |
| `description TO-Leaf3-Eth1` | `description TO-Leaf3-Eth2` | `name HOST1-NETWORK` | `name HOST2-NETWORK` | `name HOST3-HOST4-NETWORK` | |
| `no switchport` | `no switchport` | | | | |
| `ip address` | `ip address` | `interface Ethernet3` | `interface Ethernet3` | `interface Ethernet3` | |
| `no shutdown` | `no shutdown` | `description TO-Host1` | `description TO-Host2` | `description TO-Host3` | |
| | | `switchport mode access` | `switchport mode access` | `switchport mode access` | |
| `interface Loopback0` | `interface Loopback0` | `switchport access vlan 10` | `switchport access vlan 20` | `switchport access vlan 30` | |
| `ip address` | `ip address` | `no shutdown` | `no shutdown` | `no shutdown` | |
| | | | | | |
| `end` | `end` | `interface Vlan10` | `interface Vlan20` | `interface Ethernet4` | |
| `write memory` | `write memory` | `description GW-HOST1` | `description GW-HOST2` | `description TO-Host4` | |
| | | `ip address` | `ip address` | `switchport mode access` | |
| | | `no shutdown` | `no shutdown` | `switchport access vlan 30` | |
| | | | | `no shutdown` | |
| | | `interface Loopback0` | `interface Loopback0` | | |
| | | `ip address` | `ip address 10.255.0.12/32` | `interface Vlan30` | |
| | | `end` | `write memory` | `description GW-HOST3-HOST4` | |
| | | `write memory` | | `ip address` | |
| | | | | `no shutdown` | |
| | | | | | |
| | | | | `interface Loopback0` | |
| | | | | `ip address` | |
| | | | | | |
| | | | | `end` | |
| | | | | `write memory` | |
