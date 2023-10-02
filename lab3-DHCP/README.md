# Lab - Configure Router-on-a-Stick Inter-VLAN Routing

## Топология сети:

![](topology.png)

  Таблица сетевых адресов

| Device | Interface | IP Address   | Subnet Mask     |Default Gateway|
|------- |------    |---------      |---------------- |---------------|
| R1     | e0/0   | 10.0.0.1    | 255.255.255.252  | N/A			|
|        | e0/1   | N/A    | N/A  | N/A			|
|        | e0/1.100   |            |              | N/A			|
|         |  e0/1.200    |               |                   |  N/A      |
|         |  e0/1.1000 |   N/A       |              N/A   | N/A       |
| R1     | e0/0   | 10.0.0.2   | 255.255.255.252 | N/A	|
|      | e0/1   |     |  | N/A	|
|   S1   | VLAN200   |     |  | 	|
|   S2   | VLAN1   |     |  | 	|
| PC-A   | NIC      | DHCP   | DHCP   | DHCP	|
| PC-B   | NIC      | DHCP    | DHCP  | DHCP	|


## Таблица VLAN
| VLAN| NAME| Interface Assigned |
|------|----|-------------------|
|1|N/A| S1: e0/3 |
|100| Clients | S1: e0/0|
|200| Management| S1: VLAN 200|
|999|Parking_Lot|S1: e0/1-3 |
|1000| Native| N/A|