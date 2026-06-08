# Распределение адресного пронстранства 
## 1 План настройки Spine-Leaf архитектуры (BGP)

- [ ] **Подготовить адресное пространство** для Loopback-интерфейсов и point-to-point линков Spine–Leaf.
- [ ] **Назначить уникальные BGP AS number** каждому сетевому устройству.
- [ ] **Включить маршрутизацию IPv4** на каждом устройстве командой ip routing.
- [ ] **Настроить routed-интерфейсы между Spine и Leaf:**
  - [ ] Перевести Ethernet-интерфейсы в L3-режим командой no switchport.
  - [ ] Назначить IP-адреса из /31-сетей.
  - [ ] Включить интерфейсы командой no shutdown.
- [ ] Настроить Loopback0 на каждом устройстве.
- [ ] **Настроить eBGP-соседства:**
  - [ ] **Spine1** с **Leaf1**, **Leaf2**, **Leaf3**.
  - [ ] **Spine2** с **Leaf1**, **Leaf2**, **Leaf3**.
- [ ] **Анонсировать в BGP:**
  - [ ] Loopback0 каждого устройства.
  - [ ] Connected-сети Spine–Leaf.
- [ ] **Включить BGP ECMP:**
  - [ ] maximum-paths 4
  - [ ] bgp bestpath as-path multipath-relax
- [ ] **Проверить:**
  - [ ] Состояние интерфейсов.
  - [ ] Состояние BGP-соседств.
  - [ ] Наличие BGP-маршрутов.
  - [ ] Наличие маршрутов в routing table.
  - [ ] IP-связность между Loopback0 всех сетевых устройств.

## 2. Архитектура сети
Используется двухуровневая CLOS архитектура:
<img width="1361" height="1224" alt="Arch" src="https://github.com/user-attachments/assets/d8cd80b5-4d89-4028-9288-285f9fdec8ed" />

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

### 6.1 Проверка маршрутов 

Spine1 

<img width="960" height="538" alt="bgp route " src="https://github.com/user-attachments/assets/5a9c53c1-8218-4310-baf1-b320e01e9746" />


Spine2

<img width="1030" height="532" alt="bgp route" src="https://github.com/user-attachments/assets/62c67cd4-9049-4072-91b9-572255ba53d6" />

Leaf1

<img width="974" height="549" alt="bgp route" src="https://github.com/user-attachments/assets/3cc4afa0-0369-4572-a946-f6b5e0cb0f89" />

Leaf2

<img width="967" height="525" alt="bgp route" src="https://github.com/user-attachments/assets/81db4d4a-6434-4112-8c07-9c998d4648f6" />

Leaf3

<img width="1004" height="520" alt="bgp routee" src="https://github.com/user-attachments/assets/730022a3-2ae3-49b6-a4aa-98331c32e741" />

### 6.2 Ping

Spine1

<img width="610" height="575" alt="ping" src="https://github.com/user-attachments/assets/b95e1e2e-ca51-409a-8c74-ee3f166a52d0" />


Spine2

<img width="646" height="712" alt="ping" src="https://github.com/user-attachments/assets/a212c4ca-4763-4f79-bf61-c2f0d1136cbf" />


Leaf1

<img width="632" height="705" alt="ping" src="https://github.com/user-attachments/assets/1b6e277e-467b-49bf-92d8-94aa1733bc89" />

Leaf2

<img width="659" height="734" alt="ping" src="https://github.com/user-attachments/assets/cf01e7d4-aa5a-4143-abcc-1e0e8a7bcff0" />


Leaf3

<img width="606" height="705" alt="ping" src="https://github.com/user-attachments/assets/2ff1137f-a363-4e5e-bd67-fee686718170" />

