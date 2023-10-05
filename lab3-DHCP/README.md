# Lab - Configure Router-on-a-Stick Inter-VLAN Routing

## Топология сети:

![](topology.png)

  Таблица сетевых адресов

| Device | Interface | IP Address   | Subnet Mask     |Default Gateway|
|------- |------    |---------      |---------------- |---------------|
| R1     | e0/0   | 10.0.0.1    | 255.255.255.252  | N/A			|
|        | e0/1   | N/A    | N/A  | N/A			|
|        | e0/1.100   |     192.168.1.1       |    255.255.255.192          | N/A			|
|         |  e0/1.200    |        192.168.1.65       |       255.255.255.224            |  N/A      |
|         |  e0/1.1000 |   N/A       |              N/A   | N/A       |
| R2     | e0/0   | 10.0.0.2   | 255.255.255.252 | N/A	|
|      | e0/1   |   192.168.1.97  | 255.255.255.240 | N/A	|
|   S1   | VLAN200   | 192.168.1.66    | 255.255.255.224 |192.168.1.65	|
|   S2   | VLAN1   |  192.168.1.98   | 255.255.255.240 |192.168.1.97	|
| PC-A   | NIC      | DHCP   | DHCP   | DHCP	|
| PC-B   | NIC      | DHCP    | DHCP  | DHCP	|


## Таблица VLAN
| VLAN| NAME| Interface Assigned |
|------|----|-------------------|
|1|N/A| S2: e0/3 |
|100| Clients | S1: e0/0|
|200| Management| S1: VLAN 200|
|999|Parking_Lot|S1: e0/1-2 |
|1000| Native| N/A|

## Часть 1: Построение сети и настройка основных параметров устройства

### Шаг 1: Создайте схему адресации

Подсоедините сеть 192.168.1.0/24, чтобы она отвечала следующим требованиям:

1. “Подсеть A”, поддерживающая 58 хостов (клиентская VLAN на R1):
```192.168.1.0/26``` подсеть подддерживает 64 хостов самая маленькая подсеть, которая вмещает 58 хостов.
Первый IP-адрес в таблице адресации для R1 G0/0/1.100:
```192.168.1.1```

2.  “Подсеть B”, поддерживающая 28 хостов (управляющая VLAN на R1): 
```192.168.1.64/27```. Первый IP-адрес в таблице адресации для R1 G0/0/1.200: ```192.168.1.65/27```. Запишите второй IP-адрес в таблице адресов для S1 VLAN 200: ```192.168.1.66/27```и введите соответствующий шлюз по умолчанию: ```192.168.1.95/27```.

3. “Подсеть C”, поддерживающая 12 хостов (клиентская сеть на R2): ```192.168.1.96/28 ```. Запишите первый IP-адрес в таблице адресации для R2 G0/0/1: ```192.168.1.97/28```.

### Шаг 2: Подключите сеть кабелем, как показано в топологии.

### Шаг 3: Настройте основные параметры для каждого маршрутизатора.

1. Назначьте маршрутизатору имя устройства.Откройте окно конфигурации:
```
Router(config)#hostname R1
```
2. Отключите поиск по DNS, чтобы маршрутизатор не пытался перевести неправильно введенные команды так, как если бы они были именами хостов.
```
R1(config)#no ip domain-lookup 
```
3. Назначьте class в качестве привилегированного зашифрованного пароля EXEC.
```
R1(config)#enable secret class
```
4. Назначьте cisco в качестве пароля консоли и включите вход в систему.
```
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
```
5. Назначьте cisco в качестве пароля VTY и разрешите вход в систему.
```
R1(config)#line vty 0 4 
R1(config-line)#password cisco
R1(config-line)#login
```
6. Зашифруйте пароли в виде открытого текста.
```
R1(config)#service password-encryption 
```
7. Создайте баннер, предупреждающий любого, кто получает доступ к устройству, о том, что несанкционированный доступ запрещен.
```
R1(config)#banner motd #access to this device is prohibited#
```
8. Сохраните текущую конфигурацию в файле конфигурации запуска.
```
R1#wr me
*Oct  2 18:04:53.497: %SYS-5-CONFIG_I: Configured from console by console
R1#wr mem
Building configuration...
```
9. Установите часы на маршрутизаторе на сегодняшнее время и дату.
Примечание: Используйте знак вопроса (?), чтобы указать правильную последовательность параметров, необходимых для выполнения этой команды.
```
R1(config)#clock timezone CST +3
```
Аналогично с роутером R2

### Шаг 4: Настройте маршрутизацию между VLAN на R1
1. Активируйте интерфейс Et0/1 на маршрутизаторе.
```
R1(config)#int et0/1      
R1(config-if)#no shutdown 
```
2. Настройте подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q, и им присваивается первый доступный адрес из рассчитанного вами пула IP-адресов. Убедитесь, что подинтерфейсу для собственной VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.
```
R1(config-if)#int et0/1.100
R1(config-subif)#description Default Geteway for VLAN 100
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#exit
R1(config)#interface et0/1.200
R1(config-subif)#description Default Geteway for VLAN 200
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#exit
R1(config)#int et0/1.1000
R1(config-subif)#description Default Geteway for VLAN 1000
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#exit
```
3. Убедитесь, что подинтерфейсы работают.
```
R1#show ip interface brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  administratively down down    
Ethernet0/1                unassigned      YES unset  up                    up      
Ethernet0/1.100            192.168.1.1     YES manual up                    up      
Ethernet0/1.200            192.168.1.65    YES manual up                    up      
Ethernet0/1.1000           unassigned      YES unset  up                    up      
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
```


### Шаг 5: Настройте Et0/3 на R2, затем Et0/0 и статическую маршрутизацию для обоих маршрутизаторов
1. Настройте Et0/3 на R2 с первым IP-адресом подсети C, который вы вычислили ранее.
```
R2(config)#int et0/3
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no shutdown
```
2. Настройте интерфейс Et0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации.
R2:
```
R2(config)#int et0/0
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shutdown
```
R1:
```
R1(config)#int et0/0
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shutdown
```
3. Настройте маршрут по умолчанию на каждом маршрутизаторе, указанном на IP-адрес Et0/0 на другом маршрутизаторе.
```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
4. Убедитесь, что статическая маршрутизация работает, проверив адрес Et0/3 R2 от R1.
```
R1#ping 192.168.1.97       
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
5. Сохраните текущую конфигурацию в файле конфигурации запуска.
```
R1#wr mem
Building configuration...
R2#wr mem
Building configuration...
```

### Шаг 6: Настройте основные параметры для каждого коммутатора.
1. Назначьте коммутатору имя устройства.
Откройте окно конфигурации
```
Switch(config)#hostname S1
```
2. Отключите поиск по DNS, чтобы маршрутизатор не пытался перевести неправильно введенные команды так, как если бы они были именами хостов.
```
S1(config)#no ip domain-lookup 
```
3. Назначьте class в качестве привилегированного зашифрованного пароля EXEC.
```
S1(config)#enable secret class
```
4. Назначьте cisco в качестве пароля консоли и включите вход в систему.
```
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
```
5. Назначьте cisco в качестве пароля VTY и разрешите вход в систему.
```
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
```
6. Зашифруйте пароли в виде открытого текста.
```
S1(config)#service password-encryption 
```
7. Создайте баннер, предупреждающий любого, кто получает доступ к устройству, о том, что несанкционированный доступ запрещен.
```
S1(config)#banner motd #access to this device is prohibited#
```
8. Сохраните текущую конфигурацию в файле конфигурации запуска.
```
S1#wr mem
Building configuration...
Compressed configuration from 915 bytes to 661 bytes[OK]
```
9. Установите часы на переключателе на сегодняшнее время и дату.
Примечание: Используйте знак вопроса (?), чтобы указать правильную последовательность параметров, необходимых для выполнения этой команды.
```
S1(config)#clock timezone CST +3
```
10. Скопируйте текущую конфигурацию в конфигурацию запуска.
```
S1#copy running-config startup-config
Destination filename [startup-config]? 
Building configuration...
Compressed configuration from 915 bytes to 663 bytes[OK]
```

### Шаг 7: Создайте VLAN на S1.
Примечание: S2 настроен только с базовыми настройками.
1. Создайте и назовите требуемые VLAN на коммутаторе 1 из приведенной выше таблицы.
```
S1#show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
100  Clients                          active    
200  Management                       active    
999  ParkingLot                       active    
1000 Native                           active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```
2. Сконфигурируйте и активируйте интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, вычисленный ранее. Кроме того, установите шлюз по умолчанию на S1.
```
S1(config)#int vlan 200  
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#no shutdown 
S1(config-if)#exit
S1(config)#ip route 0.0.0.0 0.0.0.0 192.168.1.65
```
3. Сконфигурируйте и активируйте интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, вычисленный ранее. Кроме того, установите шлюз по умолчанию на S2.
```
S2(config)#int vlan 1
S2(config-if)#ip address 192.168.1.98 255.255.255.240
S2(config-if)#exit
S2(config)#ip route 0.0.0.0 0.0.0.0 192.168.1.97
```
4. Назначьте все неиспользуемые порты на S1 VLAN Parking_Lot, сконфигурируйте их для статического режима доступа и административно деактивируйте их. На S2 административно деактивируйте все неиспользуемые порты.
Примечание: Команда interface range полезна для выполнения этой задачи с минимальным количеством необходимых команд.
```
S1(config)#int range et0/1-2
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#shutdown 

S2(config)#int range et0/1-2
S2(config-if-range)#switchport mode access 
S2(config-if-range)#shut
```

### Шаг 8: Назначьте VLAN правильным интерфейсам коммутатора.
1. Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и сконфигурируйте их для статического режима доступа.
Откройте окно конфигурации
```
S1(config)#int et0/0
S1(config-if)#switchport mode access 
S1(config-if)#switchport access vlan 100
S1(config-if)#no shutdown
```
2. Убедитесь, что виртуальные сети назначены правильным интерфейсам.
Вопрос:
Почему интерфейс F0/5 указан в разделе VLAN 1?
```
S1(config-if)#do show vlan bri

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3
100  Clients                          active    Et0/0
200  Management                       active    
999  ParkingLot                       active    Et0/1, Et0/2
1000 Native                           active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```
### Шаг 9: Вручную сконфигурируйте интерфейс S1 Et0/3 как магистральный 802.1Q.
1. Измените режим порта коммутатора на интерфейсе для принудительного подключения к магистрали.
```
S1(config)#int et0/3
S1(config-if)#switchport trunk encapsulation dot1q 
```
2. В рамках конфигурации магистрали установите для собственной VLAN значение 1000.
```
S1(config-if)#switchport mode trunk 
S1(config-if)#switchport trunk native vlan 1000
```
3. В качестве другой части конфигурации магистрали укажите, что сетям VLAN 100, 200 и 1000 разрешено пересекать магистраль.
```
S1(config-if)#switchport trunk allowed vlan 1000,100,200
```
4. Сохраните текущую конфигурацию в файле конфигурации запуска.
```
S1#wr me
Building configuration...
Compressed configuration from 1330 bytes to 889 bytes[OK]
```
5. Проверьте состояние транкинга.
```
S1#show interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Et0/3       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/3       100,200,1000

Port        Vlans allowed and active in management domain
Et0/3       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/3       100,200,1000
```
Вопрос:
На данный момент, какой IP-адрес был бы у ПК, если бы они были подключены к сети с помощью DHCP?
На данный момент он взял бы адрес из области адресов APIPA, так как он не получит ответ от DHCP сервера

## Часть 2: Настройка и проверка двух серверов DHCPv4 на R1
В части 2 вы настроите и проверите сервер DHCPv4 на R1. Сервер DHCPv4 будет обслуживать две подсети, подсеть A и подсеть C.
### Шаг 1: Настройте R1 с помощью пулов DHCPv4 для двух поддерживаемых подсетей. Ответ приведен ниже
1. Исключите первые пять доступных адресов из каждого пула адресов.
```
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
```
2. Создайте пул DHCP (используйте уникальное имя для каждого пула).
```
R1(config)#ip dhcp pool R1_Client_LA
```
3. Укажите сеть, которую поддерживает данный DHCP-сервер.
```
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
```
4. Настройте доменное имя следующим образом ccna-lab.com
```
R1(dhcp-config)#domain-name ccna-lab.com
```
5. Настройте соответствующий шлюз по умолчанию для каждого пула DHCP.
```
R1(dhcp-config)#default-router 192.168.1.1
```
6. Настройте время аренды на 2 дня 12 часов и 30 минут.
```
R1(dhcp-config)#lease 2 12 30
```
7. Затем сконфигурируйте второй пул DHCPv4, используя имя пула R2_Client_LAN и вычисляемую сеть, маршрутизатор по умолчанию и используйте то же доменное имя и время аренды из предыдущего пула DHCP.
```
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)#ip dhcp pool R2_Client_LAN                         
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#domain-name ccna-lab.com
R1(dhcp-config)#lease 2 12 30
R1(dhcp-config)#exit
```
