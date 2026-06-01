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

<img width="1377" height="1202" alt="OSPF" src="https://github.com/user-attachments/assets/32a1836a-6bf4-49b7-bb8e-b4039b662852" />



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
### 5.1 Spine 1 (Spine 2 - аналогично)
enable
configure terminal

hostname Spine1
ip routing

interface Ethernet1
   description TO-Leaf1-Eth1
   no switchport
   ip address 192.11.1.0/31
   ip ospf area 0.0.0.0
   ip ospf network point-to-point
   mtu 1500
   no shutdown

interface Ethernet2
   description TO-Leaf2-Eth1
   no switchport
   ip address 192.11.2.0/31
   ip ospf area 0.0.0.0
   ip ospf network point-to-point
   mtu 1500
   no shutdown

interface Ethernet3
   description TO-Leaf3-Eth1
   no switchport
   ip address 192.11.3.0/31
   ip ospf area 0.0.0.0
   ip ospf network point-to-point
   mtu 1500
   no shutdown

interface Loopback0
   ip address 11.11.11.11/32
   ip ospf area 0.0.0.0

router ospf 1
   router-id 11.11.11.11
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3

end
write memory


### 5.2 Leaf1 (Остальные лифы аналогично)
enable
configure terminal

hostname Leaf1
ip routing

vlan 10
   name HOST1_NET

interface Ethernet1
   description TO-Spine1-Eth1
   no switchport
   ip address 192.11.1.1/31
   ip ospf area 0.0.0.0
   ip ospf network point-to-point
   mtu 1500
   no shutdown

interface Ethernet2
   description TO-Spine2-Eth1
   no switchport
   ip address 192.22.1.1/31
   ip ospf area 0.0.0.0
   ip ospf network point-to-point
   mtu 1500
   no shutdown

interface Ethernet3
   description TO-Host1
   switchport
   switchport mode access
   switchport access vlan 10
   spanning-tree portfast
   no shutdown

interface Vlan10
   description GW-HOST1
   ip address 192.168.1.1/24
   ip ospf area 0.0.0.0
   no shutdown

interface Loopback0
   ip address 1.1.1.1/32
   ip ospf area 0.0.0.0

router ospf 1
   router-id 1.1.1.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2

end
write memory
## 6 Добавление BFD
Делаем только на OSPF uplink-интерфейсах Spine–Leaf:
BFD: ip ospf neighbor bfd
BFD timers: bfd interval 200 min-rx 200 multiplier 3
OSPF authentication: ip ospf authentication message-digest
OSPF key: ip ospf message-digest-key 1 md5 0 OSPF_KEY_123

## 7 Диагностика 
### 7.1 Проверка OSPF-соседств 
На Spine1
<img width="637" height="143" alt="image" src="https://github.com/user-attachments/assets/b87aa3c9-b10e-49f2-90a9-6760a982def3" />
На Spine2
<img width="644" height="156" alt="Spine2_ospf" src="https://github.com/user-attachments/assets/a0f369f7-2512-4633-8ab4-a31f4a7aaae0" />
Lef1
<img width="644" height="125" alt="image" src="https://github.com/user-attachments/assets/541df297-476b-4afa-b140-d1f476e11ebd" />
Lef2 
<img width="650" height="113" alt="image" src="https://github.com/user-attachments/assets/a711013b-7173-4f4a-aaad-41c447de43da" />
Leaf3
<img width="650" height="128" alt="image" src="https://github.com/user-attachments/assets/069ca3bd-3b05-42ec-ae45-5c79c2faa421" />



