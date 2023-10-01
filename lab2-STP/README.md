# Лабораторная работа. Развертывание коммутируемой сети с резервными каналами
## Топология сети:

![](base_scheme.png)

  Таблица сетевых адресов

| Устройство | Интерфейс | IP- адрес   | Маска подсети |
|------- |------    |---------      |---------------- |
| S1     | VLAN 1   | 192.168.1.1    | 255.255.255.0  | 
| S2     | VLAN 1   | 192.168.1.2    | 255.255.255.0 | 
| S3     | VLAN 1   | 192.168.1.3    | 255.255.255.0 | 

## Часть 1:	Создание сети и настройка основных параметров устройства
### Шаг 1:	Создана сеть согласно топологии.
### Шаг 2:	Выполнил инициализацию и перезагрузку коммутаторов.
### Шаг 3:	Настройте базовые параметры каждого коммутатора.
Настройка S1 коммутатора:

1. Отключил поиск DNS:
```
S1(config)#no ip domain-lookup 
```
2. Присвоил имя устройству в соответствии с топологией:
```
Switch(config)#hostname S1
```
3. Назначил ```class``` в качестве зашифрованного пароля доступа к привилегированному режиму:
```
S1(config)#enable secret class
```
4. Назначил ```cisco``` в качестве паролей консоли и VTY и активировал вход для консоли и VTY каналов:
```
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 4 
S1(config-line)#password cisco
S1(config-line)#login 
```
5. Настроил logging synchronous для консольного канала.
```
S1(config)#line console 0
S1(config-line)#logging synchronous 
```
6. Настроил баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа:
```
S1(config)#banner motd #access to this device is prohibited#
```
7. Задал IP-адрес, указанный в таблице адресации для VLAN 1 на коммутаторе.
```
S1(config)#int vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown 
```
8. Скопировал текущую конфигурацию в файл загрузочной конфигурации:
```
S1#copy running-config startup-config
Destination filename [startup-config]? 
Building configuration...
Compressed configuration from 933 bytes to 660 bytes[OK]
```
Аналогично с другими коммутаторами 
### Шаг 4:	Проверьте связь.