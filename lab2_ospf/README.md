# Распределение адресного пронстранства 
## 1. План работ
Настроить OSPF для Underlay сети.
## 2. Архитектура сети
Используется двухуровневая CLOS архитектура:

Spine уровень:
Spine1
Spine2

Leaf уровень:
Leaf1
Leaf2
Leaf3

Хосты:
Host1
Host2
Host3
Host4
## 3. План работ
### 3.1 Собрать CLOS-схему в PNETLab:
Spine1, Spine2
Leaf1, Leaf2, Leaf3
Host1–Host4

### 3.3 Назначить IP-адреса:
/31 на point-to-point линках Spine–Leaf
/32 на Loopback0
/24 для пользовательских VLAN-сетей

### 3.4 Включить L3-маршрутизацию на Arista:
ip routing
no switchport на routed-интерфейсах
Настроить VLAN/SVI на leaf-коммутаторах.

### 3.5 Настроить OSPF underlay:
process ID: 1
area: 0.0.0.0
loopback использовать как router-id
spine-leaf интерфейсы сделать активными OSPF-интерфейсами
host-интерфейсы оставить passive

### 3.6 Проверить:
соседства OSPF
маршруты OSPF
ping между loopback-адресами
ping между host-сетями

<img width="1413" height="1192" alt="CLOS_arch" src="https://github.com/user-attachments/assets/9a204d0d-70a9-4a54-b3eb-3a0e20ecc1d4" />


## 4. Адрессное пронстранство 
### 4.1 Loopback

| Устройство | Адрес |
| :--- | :--- |
| Spine1 | 11.11.11.11/32 |
| Spine2 | 22.22.22.22/32 |
| Leaf1 | 1.1.1.1/32 |
| Leaf2 | 2.2.2.2/32 |
| Leaf3 | 3.3.3.3/32 |

### 4.2 Spine1 <-> Leaf

| Линк | Spine1 | Leaf |
| :--- | :--- | :--- |
| S1-L1 | 192.11.1.0/31 | 192.11.1.1/31 |
| S1-L2 | 192.11.2.0/31 | 192.11.2.1/31 |
| S1-L3 | 192.11.3.0/31 | 192.11.3.1/31 |

### 4.3 Spine2 <-> Leaf

| Линк | Spine2 | Leaf |
| :--- | :--- | :--- |
| S2-L1 | 192.22.1.0/31 | 192.22.1.1/31 |
| S2-L2 | 192.22.2.0/31 | 192.22.2.1/31 |
| S2-L3 | 192.22.3.0/31 | 192.22.3.1/31 |

### 4.4 Host-сети

| VLAN | Сеть | Gateway |
| :--- | :--- | :--- |
| 10 | 192.168.1.0/24 | 192.168.1.1 |
| 20 | 192.168.2.0/24 | 192.168.2.1 |
| 30 | 192.168.3.0/24 | 192.168.3.1 |

### 4.5 Hosts

| Host | IP | GW |
| :--- | :--- | :--- |
| Host1 | 192.168.1.10 | 192.168.1.1 |
| Host2 | 192.168.2.10 | 192.168.2.1 |
| Host3 | 192.168.3.10 | 192.168.3.1 |
| Host4 | 192.168.3.11 | 192.168.3.1 |

## 5. Config

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
