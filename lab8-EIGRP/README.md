## Домашнее задание
# EIGRP

## Цель:
Настроить EIGRP в С.-Петербург;
Использовать named EIGRP


## Описание/Пошаговая инструкция выполнения домашнего задания:
1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.


Топология сети:

![](base_scheme.png)

### 1. В офисе С.-Петербург настроить EIGRP.


```   
R17(config)#router eigrp 1
R17(config-router)#eigrp router-id 17.17.17.17
R17(config-router)#network 192.168.0.9 255.255.255.252
R17(config-router)#network 192.168.0.13 255.255.255.252
R17(config-router)#network 192.168.0.6 255.255.255.252
```


```
R18(config)#router eigrp 1
R18(config-router)#eigrp router-id 18.18.18.18
R18(config-router)#network 192.168.122.2 255.255.255.252
R18(config-router)#network 192.168.121.1 255.255.255.252
R18(config-router)#network 192.168.0.1 255.255.255.252
R18(config-router)#network 192.168.0.5 255.255.255.252
```

```
R16(config)#  router eigrp 1
R16(config-router)#eigrp router-id 16.16.16.16
R16(config-router)#network 192.168.0.17 255.255.255.252
R16(config-router)#network 192.168.0.2 255.255.255.252
R16(config-router)#network 192.168.0.21 255.255.255.252
R16(config-router)#network 192.168.0.26 255.255.255.252
```
```
R32(config)#router eigrp 1
R32(config-router)#eigrp router-id 32.32.32.32
R32(config-router)#network 192.168.0.25 255.255.255.252
```