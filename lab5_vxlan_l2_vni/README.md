# Распределение адресного пронстранства 
Настроить Overlay-сеть на базе VXLAN EVPN поверх существующего eBGP Underlay для обеспечения L2-связанности между клиентами, подключёнными к разным Leaf-коммутаторам.

В рамках работы реализуется:

1. eBGP Underlay между Spine и Leaf по routed /31 линкам
2. Распространение Loopback0-адресов через Underlay
3. eBGP EVPN Overlay между Leaf и Spine по Loopback0
4. Настройка VXLAN VTEP на Leaf-коммутаторах
5. Привязка VLAN к VNI
6. Проверка EVPN route-type 3 IMET
7. Проверка EVPN route-type 2 MAC/IP
8. Проверка L2-связанности между Host1 и Host2 через VXLAN

## 2. Архитектура сети
| Устройство | Роль                  | Loopback0        | AS    |
|-------------|-----------------------|------------------|-------|
| Spine1      | Underlay/EVPN transit | `11.11.11.11/32` | 65001 |
| Spine2      | Underlay/EVPN transit | `22.22.22.22/32` | 65002 |
| Leaf1       | VTEP                  | `1.1.1.1/32`     | 65101 |
| Leaf2       | VTEP                  | `2.2.2.2/32`     | 65102 |
| Leaf3       | VTEP                  | `3.3.3.3/32`     | 65103 |
Используется двухуровневая CLOS архитектура:
<img width="1016" height="803" alt="image" src="https://github.com/user-attachments/assets/42d89bee-0f9f-480a-b59f-a71a6aa115e7" />

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

