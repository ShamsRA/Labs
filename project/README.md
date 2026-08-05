# 1 Цель проектной работы

Спроектировать геораспределённую сеть двух центров обработки данных, расположенных в разных городах. Каждый ЦОД представляет собой независимую Leaf-Spine EVPN-VXLAN-фабрику. Между площадками организовать отказоустойчивое L3 DCI-соединение через два независимых маршрутизатора. Обеспечить L2-связность внутри сегментов, межсегментную маршрутизацию внутри POD и IP-маршрутизацию между двумя POD.

Результат работы

Внутри каждого POD должны работать:

eBGP Underlay;
ECMP между Leaf и Spine;
eBGP EVPN Overlay;
VXLAN;
L2VNI;
L3VNI;
Anycast Gateway;
EVPN Route Type 2, 3 и 5.

Между POD должны работать:

два независимых L3 DCI-пути;
IPv4 eBGP в VRF TENANT;
передача только клиентских IP-префиксов;
отсутствие растягивания VLAN между городами;
автоматическое переключение при отказе одного DCI-пути.

# 2. Архитектура сети

Используется двухуровневая CLOS архитектура:
<img width="2416" height="869" alt="image" src="https://github.com/user-attachments/assets/903f4d35-4400-415e-a96c-8716312eeafe" />

## 2.1 Функции устройств

| Устройство          | Назначение                                                   |
| ------------------- | ------------------------------------------------------------ |
| **Spine**           | Underlay-маршрутизация и EVPN Route Server                   |
| **Leaf**            | VTEP, шлюз VLAN, поддержка L2VNI и L3VNI                     |
| **Leaf1 / Leaf2**   | Дополнительно выполняют роль Border Leaf для организации DCI |
| **DCI-R1 / DCI-R2** | Обычная IPv4-маршрутизация между городами                    |
| **Host**            | Клиентские устройства                                        |

## 2.2 Роль DCI-R1 и DCI-R2

Маршрутизаторы **DCI-R1** и **DCI-R2**:

* не являются VTEP;
* не имеют интерфейса `Vxlan1`;
* не участвуют в EVPN;
* не передают MAC-маршруты;
* передают только IPv4-префиксы VRF `TENANT`.

## 3. План работ
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

<img width="915" height="477" alt="image" src="https://github.com/user-attachments/assets/5e3b6c89-90bb-4c92-9b86-5f2a1587af06" />



Spine2

Spine2
Routed-интерфейсы Ethernet1–Ethernet3 находятся в состоянии up/up.
Underlay eBGP-соседства со всеми Leaf находятся в состоянии Established.
Получены маршруты до Loopback0 всех Leaf.
EVPN-соседства с Leaf1, Leaf2 и Leaf3 находятся в состоянии Established.

<img width="962" height="470" alt="image" src="https://github.com/user-attachments/assets/c7959ee7-a5d6-4262-807a-e4f58460ccf4" />

Leaf1

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства со Spine1 и Spine2 находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf1 — 1.1.1.1.
VLAN10 сопоставлен с L2 VNI10010.
MAC Host1 изучен локально через Ethernet3.
MAC Host3 изучен через Vxlan1, удалённый VTEP — 3.3.3.3
<img width="920" height="456" alt="image" src="https://github.com/user-attachments/assets/2012fb45-00cb-4de3-8aef-aa5ed1316590" />


Leaf2

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf2 — 2.2.2.2.
VLAN20 сопоставлен с L2 VNI10020.
Leaf2 участвует в общей EVPN VXLAN fabric.

<img width="918" height="459" alt="image" src="https://github.com/user-attachments/assets/da318917-0b64-4ec0-ae9f-7fde29d0c586" />


Leaf3

Underlay eBGP-соседства с обоими Spine подняты.
EVPN-соседства находятся в состоянии Established.
Vxlan1 находится в состоянии up/up.
VTEP-адрес Leaf3 — 3.3.3.3.
VLAN10 сопоставлен с L2 VNI10010.
MAC Host3 изучен локально через Ethernet3.
MAC Host1 изучен через Vxlan1, удалённый VTEP — 1.1.1.1.


<img width="903" height="460" alt="image" src="https://github.com/user-attachments/assets/ecfcdd22-8b6a-4202-9a1e-b7c83c708ca7" />

Проверка маршрутизации с Host1 
<img width="517" height="671" alt="image" src="https://github.com/user-attachments/assets/ae36575b-2eca-49cf-a772-326dae4a2ff5" />


ping 192.168.2.10
ping 192.168.3.10
ping 192.168.4.10

С Host 2 
<img width="516" height="657" alt="image" src="https://github.com/user-attachments/assets/4fd2592a-4124-40be-898c-c917b0d65756" />

ping 192.168.1.10
ping 192.168.3.10
ping 192.168.4.10

С Host3:

<img width="531" height="496" alt="image" src="https://github.com/user-attachments/assets/32e5a63b-826e-4d86-811d-f245003dc203" />

ping 192.168.1.10
ping 192.168.2.10
ping 192.168.4.10



### 6.2 Проверка EVPN Overlay

Spine 1 

<img width="907" height="131" alt="image" src="https://github.com/user-attachments/assets/08186a70-fe5a-4596-ab8f-a66a4e77f32d" />

Spine 2

<img width="902" height="127" alt="image" src="https://github.com/user-attachments/assets/e3879dc9-3f4c-4401-9f72-0feef7443de3" />

Leaf 1

<img width="897" height="116" alt="image" src="https://github.com/user-attachments/assets/4b8c1dbe-2834-41ca-bd8d-9a4b2adb5316" />

Leaf 2

<img width="915" height="123" alt="image" src="https://github.com/user-attachments/assets/a09fb85f-ec60-4dcb-9f65-6ab403a007f8" />

Leaf 3

<img width="907" height="124" alt="image" src="https://github.com/user-attachments/assets/3d355c0c-7000-450f-ae1c-edc738b53a85" />

### 6.3 EVPN Route type и TENANT
Leaf 1

<img width="890" height="709" alt="image" src="https://github.com/user-attachments/assets/023a5c75-f4d5-4fac-bb82-a7385ad03c17" />

Leaf 2 

<img width="889" height="722" alt="image" src="https://github.com/user-attachments/assets/9c8c3796-b840-433a-a16c-cbf2a12702c3" />

Leaf 3

<img width="893" height="677" alt="image" src="https://github.com/user-attachments/assets/4ef0e4af-4e8a-4753-b5fd-b928bec3ba95" />






