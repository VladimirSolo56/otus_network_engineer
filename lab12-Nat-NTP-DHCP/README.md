# Домашнее задание
# Основные протоколы сети интернет

## Цель:
1. Настроить DHCP в офисе Москва
2. Настроить синхронизацию времени в офисе Москва
3. Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах


## Описание/Пошаговая инструкция выполнения домашнего задания:

1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
3. Настроите статический NAT для R20.
4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
5. *  Настроите статический NAT(PAT) для офиса Чокурдах.
6. Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
7. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.
8. Все офисы в лабораторной работе должны иметь IP связность.
 План работы и изменения зафиксированы в документации.
Документация оформлена на github. (желательно использовать markdown).
Если нужна помощь - пишите через ЛК с помощью кнопки "чат с преподавателем" или в канал в Telegram.

## 1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.

1. Для трансляции необходимо определить список контроля доступа, разрешающий адреса, которые должны быть преобразованы:
```
R14(config)#access-list 5 permit 192.168.14.0 0.0.0.255
```
```
R15(config)#access-list 5 permit 192.168.14.0 0.0.0.255
```
2. Настроим динамическое преобразование адреса источника, указав параметры списка контроля доступа, выходящий интерфейс и перегрузки:
```
R14(config)#ip nat inside source list 5 interface ethernet 0/2 overload
```
```
R15(config)#ip nat inside source list 5 interface ethernet 0/2 overload
```

3. Определим внутреннии интерфейсы:
```
R14(config)#interface range et 0/0-1, et0/3,et1/0
R14(config-if-range)#ip nat inside
R14(config-if-range)#exit
```
```
R15(config)#interface range et 0/0-1, et0/3,et1/0
R15(config-if-range)#ip nat inside
R15(config-if-range)#exit
```
4. Определим внешний интерфейс:

```
R14(config)#int et 0/2
R14(config-if)#ip nat outside
R14(config-if)#exit
```
```
R15(config)#int et 0/2
R15(config-if)#ip nat outside
R15(config-if)#exit
```

5. Провериим работу PAT. Для этого с помощью утилиты ```ping``` с R19 пинганул R22 - Киторн и с R20 пинганул R21 - Ламас, после чего можно посмотреть таблицу NAT :
```
R14# sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 192.168.101.1:3   192.168.14.10:3    192.168.101.2:3    192.168.101.2:3
```
```
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 192.168.111.2:0   192.168.14.22:0    192.168.111.1:0    192.168.111.1:0

```
Все работает