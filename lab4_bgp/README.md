# Распределение адресного пронстранства 
## 1. План работ
Настроить eBGP для Underlay сети.
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

### 5.2 Leaf1 (Остальные лифы аналогично)


## 6 Диагностика 

### 6.1 

Spine1 



Spine2



Leaf1

af2



Leaf 3




### 6.2 Проверка маршрутов 

Spine1 



Spine2


Leaf1


Leaf2




### 6.3 LSDB

Spine1



Spine2


Leaf1



Leaf2 



Leaf3



### 6.4 Ping

Spine1


Spine2


Leaf1


Leaf2



Leaf3

