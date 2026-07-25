# Распределение адресного пронстранства 
Настроить Overlay-сеть на базе VXLAN EVPN поверх существующего eBGP Underlay для обеспечения L2-связанности между клиентами, подключёнными к разным Leaf-коммутаторам.

В рамках работы предстоит:

- Сохранить существующую Leaf–Spine архитектуру и eBGP Underlay.
- Создать отдельный VLAN и L2VNI для каждого клиента.
- Создать VRF TENANT на Leaf-коммутаторах.
- Настроить SVI — шлюзы клиентских сетей.
- Связать VRF TENANT с L3VNI 5000.
- Настроить распространение клиентских сетей через BGP EVPN.
- Проверить состояние EVPN-соседств.
- Проверить соответствие VLAN, L2VNI и L3VNI.
- Проверить таблицу маршрутизации VRF.
- Проверить связность между всеми клиентами.


## 2. Архитектура сети
| Устройство | Роль                  | Loopback0        | AS    |
|-------------|-----------------------|------------------|-------|
| Spine1      | Underlay/EVPN transit | `11.11.11.11/32` | 65001 |
| Spine2      | Underlay/EVPN transit | `22.22.22.22/32` | 65002 |
| Leaf1       | VTEP                  | `1.1.1.1/32`     | 65101 |
| Leaf2       | VTEP                  | `2.2.2.2/32`     | 65102 |
| Leaf3       | VTEP                  | `3.3.3.3/32`     | 65103 |
Используется двухуровневая CLOS архитектура:
<img width="1090" height="952" alt="image" src="https://github.com/user-attachments/assets/2f6b4843-0fa2-49b2-aa99-65f0459301d3" />

## Underlay

| Устройство | Loopback0 | AS |
| :--- | :--- | :--- |
| Spine1 | 11.11.11.11/32 | 65001 |
| Spine2 | 22.22.22.22/32 | 65002 |
| Leaf1 | 1.1.1.1/32 | 65101 |
| Leaf2 | 2.2.2.2/32 | 65102 |
| Leaf3 | 3.3.3.3/32 | 65103 |

## Клиентские сети

| Клиент | Подключение | VLAN | L2VNI | IP-адрес | Шлюз |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Host1 | Leaf1 Ethernet3 | 10 | 10010 | 192.168.1.10/24 | 192.168.1.1 |
| Host2 | Leaf2 Ethernet3 | 20 | 10020 | 192.168.2.10/24 | 192.168.2.1 |
| Host3 | Leaf3 Ethernet3 | 30 | 10030 | 192.168.3.10/24 | 192.168.3.1 |
| Host4 | Leaf3 Ethernet4 | 40 | 10040 | 192.168.4.10/24 | 192.168.4.1 |

## L3 Overlay

| Параметр | Значение |
| :--- | :--- |
| VRF | TENANT |
| L3VNI | 5000 |
| Route Target | 5000:5000 |
| Virtual-router MAC | 00:1c:73:00:00:01 |

---
*Источник данных: предоставленная вами информация (применённые конфигурации Leaf1–Leaf3 из текущей лабораторной работы).*
## 3. План работ
### 3.1 Топология
2×Spine (S1, S2), 3×Leaf (L1–L3), 4×Host.
Каждый leaf подключен к обоим spine (CLOS).

### 3.2 Адресация
P2P линкам: /31
Loopback: /32
Хостовые VLAN: /24

Underlay links
| Линк         |           Spine |            Leaf |
| ------------ | --------------: | --------------: |
| Spine1–Leaf1 | `192.11.1.0/31` | `192.11.1.1/31` |
| Spine1–Leaf2 | `192.11.2.0/31` | `192.11.2.1/31` |
| Spine1–Leaf3 | `192.11.3.0/31` | `192.11.3.1/31` |
| Spine2–Leaf1 | `192.22.1.0/31` | `192.22.1.1/31` |
| Spine2–Leaf2 | `192.22.2.0/31` | `192.22.2.1/31` |
| Spine2–Leaf3 | `192.22.3.0/31` | `192.22.3.1/31` |



### 3.5 Проверка
Underlay
show ip bgp summary
show ip route 1.1.1.1
show ip route 2.2.2.2
show ip route 3.3.3.3

EVPN Overlay
show bgp evpn summary

VXLAN
show interfaces vxlan 1
show vxlan vni

MAC/IP routes
show bgp evpn route-type mac-ip

## 5. Config
### 5.1 Spine 1 (Spine 2 - аналогично)

### 5.2 Leaf1 (Остальные лифы аналогично)


## 6 Диагностика 

### 6.1 Проверка маршрутов 

Spine1 

Spine1
Underlay eBGP-соседства с Leaf1, Leaf2 и Leaf3 находятся в состоянии Established.
Маршруты до Loopback0 всех Leaf получены через BGP.
EVPN-соседства с 1.1.1.1, 2.2.2.2 и 3.3.3.3 находятся в состоянии Established.
Spine1 выполняет роль транзитного узла Underlay и EVPN control-plane.

<img width="724" height="332" alt="image" src="https://github.com/user-attachments/assets/695be591-a2cf-40a4-86bc-b2338efbdeb1" />


Spine2

Spine2
Routed-интерфейсы Ethernet1–Ethernet3 находятся в состоянии up/up.
Underlay eBGP-соседства со всеми Leaf находятся в состоянии Established.
Получены маршруты до Loopback0 всех Leaf.
EVPN-соседства с Leaf1, Leaf2 и Leaf3 находятся в состоянии Established.

<img width="705" height="329" alt="image" src="https://github.com/user-attachments/assets/57272ada-c27f-45e8-8909-b95113cb78bd" />

Leaf1

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства со Spine1 и Spine2 находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf1 — 1.1.1.1.
VLAN10 сопоставлен с L2 VNI10010.
MAC Host1 изучен локально через Ethernet3.
MAC Host3 изучен через Vxlan1, удалённый VTEP — 3.3.3.3.

<img width="996" height="830" alt="image" src="https://github.com/user-attachments/assets/1d97b2c1-6c97-4155-8b00-99b916389da2" />


Leaf2

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf2 — 2.2.2.2.
VLAN20 сопоставлен с L2 VNI10020.
Leaf2 участвует в общей EVPN VXLAN fabric.

<img width="897" height="615" alt="image" src="https://github.com/user-attachments/assets/c11a44c6-6f3f-4751-8bae-0e47afd6ec03" />


Leaf3

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf3 — 3.3.3.3.
VLAN10 сопоставлен с L2 VNI10010.
MAC Host3 изучен локально через Ethernet3.
MAC Host1 изучен через Vxlan1, удалённый VTEP — 1.1.1.1.


<img width="936" height="830" alt="image" src="https://github.com/user-attachments/assets/1045fbe0-bf3b-4e02-a7e9-59508b19e7ae" />

Host1 
Проверка клиентской связности

Для проверки L2 VXLAN используются клиенты в одном VLAN и одной IP-подсети, но на разных Leaf:

Host1 → Leaf1 → VLAN10 → 192.168.1.10/24
Host3 → Leaf3 → VLAN10 → 192.168.1.30/24

С Host1:

ping 192.168.1.30
<img width="624" height="270" alt="image" src="https://github.com/user-attachments/assets/f2749135-0304-4c97-8027-20acdc242154" />

С Host3:

ping 192.168.1.10

Успешный ping с 0% packet loss подтверждает прохождение трафика по пути:

Host1 → Leaf1 → VNI10010 → VXLAN Underlay →
Leaf3 → VLAN10 → Host3
<img width="548" height="196" alt="image" src="https://github.com/user-attachments/assets/eaf476ea-f24f-47f6-845a-a816239c3638" />


