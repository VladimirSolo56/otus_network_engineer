# Домашнее задание
# BGP. Фильтрация

# Цель:
1. Настроить фильтрацию для офисе Москва
2. Настроить фильтрацию для офисе С.-Петербург


## Описание/Пошаговая инструкция выполнения домашнего задания:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
5. Все сети в лабораторной работе должны иметь IP связность.
6. План работы и изменения зафиксированы в документации.


Топология сети:

![](base_scheme.png)


## 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).

1. Для того, чтобы настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path). Необходимо создать ```route-map``` с регулярным выражением, которое не пропускает транзитный трафик.
На R14:
```
R14(config)#ip as-path access-list 1 permit ^$
R14(config)#route-map R14 permit 10
R14(config-route-map)#match as-path 1
R14(config-route-map)#exit
R14(config)#route-map R14 deny 65000
```
На R15:
```
R15(config)#ip as-path access-list 1 permit  ^$
R15(config)#route-map R15 permit 10
R15(config-route-map)#match as-path 1
R15(config-route-map)#exit
R15(config)#route-map R15 deny 65000
R15(config-route-map)#exit
```

2. Теперь надо закрепить за соседом в eBGP.
На R14:
```
R14(config)#router bgp 64555
R14(config-router)#neighbor 192.168.101.2 route-map R14 out
```
На R15:
```
R15(config)#router bgp 64555
R15(config-router)#neighbor 192.168.111.1 route-map R15 out
```
3. Проверить настройку удастся лишь при наличии транзитного трафика, который надо спровоцировать.

## 2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).

1. В данной топологии офисов в офисе С.-Петербург транзитного трафика не может быть, так как он подключен к одной AS BGP, но все равно стоит прописать на будующее на R18 ```route-map``` с регулярным выражением, которое не пропускает транзитный трафик:
```
R18(config)#ip as-path access-list 1 permit ^$
R18(config)#route-map R18 permit 10
R18(config-route-map)#match as-path 1
R18(config-route-map)#exit
R18(config)#route-map R18 deny 65000
R18(config-route-map)#exit
```
2. Теперь надо закрепить за соседом в eBGP.
```
R18(config)#router bgp 64955
R18(config-router)# neighbor 192.168.121.2 route-map R18 out
R18(config-router)#neighbor 192.168.122.1 route-map R18 out
R18(config-router)#end
```
3. Проверить настройку удастся лишь при наличии транзитного трафика, который надо спровоцировать.

## 3. Настроить провайдера Киторн так, чтобы в офис Москвы отдавался только маршрут по умолчанию.

1. Пропишим маршрут по умолчанию в офис Москвы на R22:
```
R22(config)#router bgp 64655
R22(config-router)#neighbor 192.168.101.1 default-originate
```

2. Проверим:
```
BGP table version is 11, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          192.168.101.2                  50      0 64655 i
 *>i 192.168.0.0      192.168.111.1            0    100      0 64755 64855 64955 i
 *                    192.168.101.2                  50      0 64655 64855 64955 i
 * i 192.168.14.0     15.15.15.15              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 192.168.28.0/30  192.168.111.1            0    100      0 64755 64855 i
 *                    192.168.101.2                  50      0 64655 64855 i
 *>i 192.168.127.0/30 192.168.111.1            0    100      0 64755 64855 i
 *                    192.168.101.2                  50      0 64655 64855 i
 *>i 192.168.129.0/30 192.168.111.1            0    100      0 64755 64855 i
 *                    192.168.101.2                  50      0 64655 64855 i
```
3. Видно, что в офис Москвы отдавался не только маршрут по умолчанию. Добавим  ```route-map``` на out, чтобы убрать остальные.
```
R22(config)#ip prefix-list 1 seq 5 permit 0.0.0.0/0
R22(config)#route-map R22-default-route permit 5
R22(config-route-map)#match ip address prefix-list 1
R22(config-route-map)#exit
R22(config)#route-map R22-default-route deny 65000
R22(config-route-map)#end
```
4. Добавим на выход к соседу в офисе Москвы R14:
```
R22(config)#router bgp 64655
R22(config-router)#neighbor 192.168.101.1 route-map R22-default-route out
R22(config-router)#end
```

5. Проверка:
R14:
```
R14#sh ip bgp
BGP table version is 11, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          192.168.101.2                  50      0 64655 i
 *>i 192.168.0.0      192.168.111.1            0    100      0 64755 64855 64955 i
 * i 192.168.14.0     15.15.15.15              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 192.168.28.0/30  192.168.111.1            0    100      0 64755 64855 i
 *>i 192.168.127.0/30 192.168.111.1            0    100      0 64755 64855 i
 *>i 192.168.129.0/30 192.168.111.1            0    100      0 64755 64855 i
```
R15:
```
R15#sh ip bgp
BGP table version is 8, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          192.168.101.2            0     50      0 64655 i
 *>  192.168.0.0      192.168.111.1                          0 64755 64855 64955 i
 * i 192.168.14.0     14.14.14.14              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  192.168.28.0/30  192.168.111.1                          0 64755 64855 i
 *>  192.168.127.0/30 192.168.111.1                          0 64755 64855 i
 *>  192.168.129.0/30 192.168.111.1                          0 64755 64855 i
```
Видим только один маршрут по умолчанию как на R14, так и на R15.
## 4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
1. Пропишим маршрут по умолчанию в офис Москвы на R21:
```
R21(config)#router bgp 64755
R21(config-router)#neighbor 192.168.111.2 default-originate
R21(config-router)#exit
```
2. Добавим  ```route-map``` на out, чтобы убрать все префиксы кроме префикса офиса С.-Петербург и маршрута по умолчанию.
```
R21(config)#ip prefix-list 1 seq 5 permit 0.0.0.0/0
R21(config)#ip prefix-list 1 seq 10 permit 192.168.0.0/24
R21(config)#route-map R21-def-rout-pref-piter permit 5
R21(config-route-map)#match ip address prefix-list 1
R21(config-route-map)#exit
R21(config)#route-map R21-def-rout-pref-piter deny 65000
R21(config-route-map)#exit
```
3. Добавим на выход к соседу в офисе Москвы R15:
```
R21(config)#router bgp 64755
R21(config-router)#neighbor 192.168.111.2 route-map R21-def-rout-pref-piter out
R21(config-router)#end
```
Проверка:
R15:
```
R15#sh ip bgp
BGP table version is 12, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          192.168.111.1                          0 64755 i
 *>  192.168.0.0      192.168.111.1                          0 64755 64855 64955 i
 * i 192.168.14.0     14.14.14.14              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
```
R14:
```
R14#sh ip bgp
BGP table version is 15, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          192.168.111.1            0    100      0 64755 i
 r                    192.168.101.2                  50      0 64655 i
 *>i 192.168.0.0      192.168.111.1            0    100      0 64755 64855 64955 i
 * i 192.168.14.0     15.15.15.15              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
```
Видим только маршрут по умолчанию и префикс офиса С.-Петербург как на R15, так и на R14.
## 5. Все сети в лабораторной работе должны иметь IP связность.
1. Проверим связность с R15:
```
R15#ping 192.168.129.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.129.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
.....
Success rate is 0 percent (0/5)
```
2. Пинг не проходит. Надо проверить статическую маршрутизацию на R15:
```
R15#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*    0.0.0.0/0 is directly connected, Null0
      14.0.0.0/32 is subnetted, 1 subnets
O        14.14.14.14 [110/21] via 192.168.14.18, 01:48:09, Ethernet0/1
                     [110/21] via 192.168.14.14, 01:48:09, Ethernet0/0
      15.0.0.0/32 is subnetted, 1 subnets
C        15.15.15.15 is directly connected, Loopback0
B     192.168.0.0/24 [20/0] via 192.168.111.1, 01:47:44
      192.168.14.0/24 is variably subnetted, 16 subnets, 3 masks
S        192.168.14.0/24 is directly connected, Null0
O        192.168.14.0/30 [110/20] via 192.168.14.18, 01:48:09, Ethernet0/1
O        192.168.14.4/30 [110/20] via 192.168.14.14, 01:48:19, Ethernet0/0
O IA     192.168.14.8/30 [110/20] via 192.168.14.41, 01:48:19, Ethernet1/0
C        192.168.14.12/30 is directly connected, Ethernet0/0
L        192.168.14.13/32 is directly connected, Ethernet0/0
C        192.168.14.16/30 is directly connected, Ethernet0/1
L        192.168.14.17/32 is directly connected, Ethernet0/1
C        192.168.14.20/30 is directly connected, Ethernet0/3
L        192.168.14.21/32 is directly connected, Ethernet0/3
O        192.168.14.24/30 [110/20] via 192.168.14.18, 01:48:09, Ethernet0/1
O        192.168.14.28/30 [110/20] via 192.168.14.18, 01:48:09, Ethernet0/1
O        192.168.14.32/30 [110/20] via 192.168.14.14, 01:48:19, Ethernet0/0
O        192.168.14.36/30 [110/20] via 192.168.14.14, 01:48:19, Ethernet0/0
C        192.168.14.40/30 is directly connected, Ethernet1/0
L        192.168.14.42/32 is directly connected, Ethernet1/0
      192.168.101.0/30 is subnetted, 1 subnets
O        192.168.101.0 [110/20] via 192.168.14.41, 01:48:19, Ethernet1/0
      192.168.111.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.111.0/30 is directly connected, Ethernet0/2
L        192.168.111.2/32 is directly connected, Ethernet0/2

```
3. Надо ее удалить, так как она не дает отправлять на Ламас и проверить заново:

```
R15(config)#no ip route 0.0.0.0 0.0.0.0 Null0
```
```
R15#ping 192.168.129.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.129.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 192.168.127.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.127.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 192.168.28.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.28.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 192.168.14.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.14.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 192.168.0.1 source 192.168.14.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.21
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
4. Пинг проходит с R15, проверим на R14 статический маршрут, удалим в случае его существовании и проверим ip связность на R14:
```
R14(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*    0.0.0.0/0 is directly connected, Null0
      14.0.0.0/32 is subnetted, 1 subnets
C        14.14.14.14 is directly connected, Loopback0
      15.0.0.0/32 is subnetted, 1 subnets
O        15.15.15.15 [110/21] via 192.168.14.6, 01:58:37, Ethernet0/1
                     [110/21] via 192.168.14.2, 01:58:27, Ethernet0/0
B     192.168.0.0/24 [200/0] via 192.168.111.1, 01:58:02
      192.168.14.0/24 is variably subnetted, 16 subnets, 3 masks
S        192.168.14.0/24 is directly connected, Null0
C        192.168.14.0/30 is directly connected, Ethernet0/0
L        192.168.14.1/32 is directly connected, Ethernet0/0
C        192.168.14.4/30 is directly connected, Ethernet0/1
L        192.168.14.5/32 is directly connected, Ethernet0/1
C        192.168.14.8/30 is directly connected, Ethernet0/3
L        192.168.14.9/32 is directly connected, Ethernet0/3
O        192.168.14.12/30 [110/20] via 192.168.14.6, 01:58:37, Ethernet0/1
O        192.168.14.16/30 [110/20] via 192.168.14.2, 01:58:27, Ethernet0/0
O IA     192.168.14.20/30 [110/20] via 192.168.14.42, 01:58:37, Ethernet1/0
O        192.168.14.24/30 [110/20] via 192.168.14.2, 01:58:27, Ethernet0/0
O        192.168.14.28/30 [110/20] via 192.168.14.2, 01:58:27, Ethernet0/0
O        192.168.14.32/30 [110/20] via 192.168.14.6, 01:58:37, Ethernet0/1
O        192.168.14.36/30 [110/20] via 192.168.14.6, 01:58:37, Ethernet0/1
C        192.168.14.40/30 is directly connected, Ethernet1/0
L        192.168.14.41/32 is directly connected, Ethernet1/0
      192.168.101.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.101.0/30 is directly connected, Ethernet0/2
L        192.168.101.1/32 is directly connected, Ethernet0/2
      192.168.111.0/30 is subnetted, 1 subnets
O        192.168.111.0 [110/20] via 192.168.14.42, 01:58:37, Ethernet1/0
```
```
R14(config)#no ip route 0.0.0.0 0.0.0.0 Null0
```
```
R14#ping 192.168.129.1 source 192.168.14.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.129.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R14#ping 192.168.127.1 source 192.168.14.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.127.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R14#ping 192.168.28.1 source 192.168.14.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.28.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R14#ping 192.168.14.21 source 192.168.14.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.14.21, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R14#ping 192.168.0.1 source 192.168.14.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.14.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
5. Все сети в лабораторной работе имеют IP связность.