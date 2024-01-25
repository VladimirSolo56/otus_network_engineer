## Домашнее задание
### PBR

#### Цель:
Настроить политику маршрутизации в офисе Чокурдах
Распределить трафик между 2 линками


### Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите политику маршрутизации для сетей офиса.
2. Распределите трафик между двумя линками с провайдером.
3. Настроите отслеживание линка через технологию IP SLA.(только для IPv4)
4. Настройте для офиса Лабытнанги маршрут по-умолчанию.
5. План работы и изменения зафиксированы в документации .
6. Документация оформлена на github. (желательно использовать markdown).

## Настроите политику маршрутизации для сетей офиса.

1. Прописал маршруты для того, чтобы проходил ```ping```. Как на switch, так и на router:
```
SW29(config)#ip route 0.0.0.0 0.0.0.0 192.168.28.1
```
```
R28(config)#ip route  10.0.30.0 255.255.255.248 192.168.28.2
```
2. Добавил ```ip address``` для интерфейса switch:
```
SW29(config)#int ethernet 0/2
SW29(config-if)#no switchport
SW29(config-if)#ip address 192.168.28.2 255.255.255.248
```
3. Проверил работоспособность офиса:
```
VPC30> ping 192.168.28.1 

84 bytes from 192.168.28.1 icmp_seq=1 ttl=254 time=0.486 ms
84 bytes from 192.168.28.1 icmp_seq=2 ttl=254 time=0.453 ms
84 bytes from 192.168.28.1 icmp_seq=3 ttl=254 time=0.440 ms
84 bytes from 192.168.28.1 icmp_seq=4 ttl=254 time=0.560 ms
84 bytes from 192.168.28.1 icmp_seq=5 ttl=254 time=0.433 ms
```
## Распределите трафик между двумя линками с провайдером.

1. Определил расширенный access-list:
```
R28(config)#ip access-list extended R25-priority-ONE
R28(config-ext-nacl)#permit tcp any any eq 22 telnet
```
2. Создал route-map:
```
R28(config)#route-map Eth0/1 permit 10
R28(config-route-map)#match ip address R25-priority-ONE
```
3. Зафиксировал на интерфейсе:
```
R28(config)#interface et0/0
R28(config-if)#ip policy route-map Eth0/1
```
PBR - работает
```
R28#show route-map
route-map Eth0/1, permit, sequence 10
  Match clauses:
    ip address (access-lists): R25-priority-ONE 
  Set clauses:
  Policy routing matches: 8 packets, 480 bytes
```
Аналогично на R25 роутере

## Настроите отслеживание линка через технологию IP SLA.(только для IPv4)

1.

##  Настройте для офиса Лабытнанги маршрут по-умолчанию.

1. Добавлил на роутер офиса Лабытнанги маршрут по-умолчанию.
```
R27(config)#ip route 0.0.0.0 0.0.0.0 192.168.127.1
```