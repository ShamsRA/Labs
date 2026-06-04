# Распределение адресного пронстранства 
## 1. План работ
Настроить ISIS для Underlay сети.
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
### 3.1 Топология
2×Spine (S1, S2), 3×Leaf (L1–L3), 4×Host.
Каждый leaf подключен к обоим spine (CLOS).

### 3.2 Адресация
P2P линкам: /31
Loopback: /32
Хостовые VLAN: /24

### 3.3 Базовая L3-настройка
ip routing
Аплинки: no switchport
SVI на leaf

### 3.4 IS-IS underlay
Процесс: UNDERLAY
Тип: level-2
Обязательно: address-family ipv4 unicast
Интерфейсы spine↔leaf: isis network point-to-point
Включение на интерфейсах: isis enable UNDERLAY
Анонс сетей: redistribute connected

### 3.5 Проверка
show isis neighbors
show ip route isis
Ping loopback↔loopback, host↔host

<img width="1439" height="1220" alt="Clos_arch" src="https://github.com/user-attachments/assets/f995e74a-963d-4300-bbc0-2e9dc1dbded9" />




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
conf t

hostname Spine1
ip routing

router isis UNDERLAY
   net 49.0001.0111.1111.1111.00
   is-type level-2
   address-family ipv4 unicast
      redistribute connected

interface Ethernet1
   no switchport
   ip address 192.11.1.0/31
   isis enable UNDERLAY
   isis network point-to-point
   no shut

interface Ethernet2
   no switchport
   ip address 192.11.2.0/31
   isis enable UNDERLAY
   isis network point-to-point
   no shut

interface Ethernet3
   no switchport
   ip address 192.11.3.0/31
   isis enable UNDERLAY
   isis network point-to-point
   no shut

interface Loopback0
   ip address 11.11.11.11/32
   isis enable UNDERLAY

end
wr

### 5.2 Leaf1 (Остальные лифы аналогично)
conf t

hostname Leaf1
ip routing

router isis UNDERLAY
   net 49.0001.0001.0001.0001.00
   is-type level-2
   address-family ipv4 unicast
      redistribute connected

interface Ethernet1
   no switchport
   ip address 192.11.1.1/31
   isis enable UNDERLAY
   isis network point-to-point

interface Ethernet2
   no switchport
   ip address 192.22.1.1/31
   isis enable UNDERLAY
   isis network point-to-point

vlan 10

interface Ethernet3
   switchport access vlan 10

interface Vlan10
   ip address 192.168.1.1/24
   isis enable UNDERLAY

interface Loopback0
   ip address 1.1.1.1/32
   isis enable UNDERLAY

end
wr

   no passive-interface Ethernet2
   no passive-interface Ethernet3

end
write memory

## 6 Диагностика 

### 6.1 Проверка соседства ISIS

Spine1 

<img width="876" height="101" alt="image" src="https://github.com/user-attachments/assets/c2ff7328-f1dc-4b25-845d-ddede7fb25c7" />

Spine2

<img width="673" height="158" alt="image" src="https://github.com/user-attachments/assets/2c9cbaa3-3fb8-4a8f-a6b9-6eba747dc358" />

Leaf1

<img width="876" height="86" alt="image" src="https://github.com/user-attachments/assets/5b9e2c38-c483-45a9-b3e8-cedcd20e03f1" />

Leaf2

<img width="875" height="85" alt="image" src="https://github.com/user-attachments/assets/d0352317-362d-442c-b143-11365044f471" />

Leaf 3

<img width="847" height="94" alt="image" src="https://github.com/user-attachments/assets/acaae2ca-a7c2-4a75-8842-a6f2fd8f0373" />


### 6.2 Проверка маршрутов 

Spine1 

<img width="511" height="440" alt="image" src="https://github.com/user-attachments/assets/d63f935d-1209-49b7-b31e-9e9953add999" />

Spine2

<img width="541" height="445" alt="image" src="https://github.com/user-attachments/assets/a02f5507-2ba6-416e-8c99-9b54ea8aebd9" />

Leaf1

<img width="528" height="475" alt="image" src="https://github.com/user-attachments/assets/9fa13eea-c462-4e8b-9c48-78b97a8f0bd2" />

Leaf2

<img width="660" height="481" alt="image" src="https://github.com/user-attachments/assets/ca675240-9e60-40dc-8d33-77dfc3691504" />

Leaf3

<img width="538" height="472" alt="image" src="https://github.com/user-attachments/assets/882c2359-24ba-4fdd-997f-9b28b2d6360c" />


### 6.3 LSDB

Spine1

<img width="536" height="177" alt="image" src="https://github.com/user-attachments/assets/e55df97e-b03d-40a2-a923-37f1f0a455cd" />


Spine2

<img width="534" height="178" alt="image" src="https://github.com/user-attachments/assets/e88b2830-6715-410d-9dd3-21c83bb00a62" />

Leaf1

<img width="523" height="181" alt="image" src="https://github.com/user-attachments/assets/589e13ad-6b14-456b-917f-090ba9ad1712" />


Leaf2 

<img width="540" height="187" alt="image" src="https://github.com/user-attachments/assets/8d867c3f-d906-4b9d-bc1a-b0ce048e8576" />


Leaf3

<img width="516" height="196" alt="image" src="https://github.com/user-attachments/assets/e6f27634-cf95-44cb-9d1a-a41967387502" />


### 6.4 Ping

Spine1

<img width="700" height="766" alt="image" src="https://github.com/user-attachments/assets/0ff146e4-9570-4775-8571-c31ca92699b2" />

Spine2

<img width="645" height="768" alt="image" src="https://github.com/user-attachments/assets/f0cf33fe-d855-4970-ba13-bd928ea74847" />

Leaf1

<img width="618" height="722" alt="image" src="https://github.com/user-attachments/assets/4edb1ce0-4ddb-424a-b878-09c18e5a32cd" />

Leaf2

<img width="774" height="730" alt="image" src="https://github.com/user-attachments/assets/5e431523-dc65-4e03-a71e-b7d3d3883dbd" />


Leaf3

<img width="747" height="755" alt="image" src="https://github.com/user-attachments/assets/f9a6ec2e-96df-4e0b-a0c4-f6076456e049" />
