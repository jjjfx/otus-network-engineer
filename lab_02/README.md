## Настройка расширенных сетей VLAN, VTP и DTP

###  Задачи:

    1. Настройка VTP
    2. Настройка DTP
    3. Добавление сетей VLAN и назначение портов
    4. Настройка расширенной сети VLAN

### Топология:
![](images/otus-vtp.png)

> Работа выполнена на оборудовании Termilab

### Таблица адресации:

| Устройство | Интерфейс     | IP-адрес     | Маска подсети   |
|:-----------|:--------------|:-------------|:----------------|
| S1         | VLAN 99       | 192.168.99.1 | 255.255.255.0   |
| S2         | VLAN 99       | 192.168.99.2 | 255.255.255.0   |
| S3         | VLAN 99       | 192.168.99.3 | 255.255.255.0   |
| PC-A       | NIC           | 192.168.10.1 | 255.255.255.0   |
| PC-B       | NIC           | 192.168.20.1 | 255.255.255.0   |
| PC-C       | NIC           | 192.168.10.2 | 255.255.255.0   |

## Часть 1. Настройка VTP

Предварительно приводим топологию коммутаторов в Termilab к указанной выше:
```
S1#sh int descr | i up
Fa0/1                          up             up       -- to S3
Fa0/3                          up             up       -- to S2
Fa0/15                         up             up       -- to PC-A

S2#sh int descr | i up
Fa0/1                          up             up       -- to S3
Fa0/3                          up             up       -- to S1
Fa0/15                         up             up       -- to PC-B

S3#sh int descr | i up
Fa0/1                          up             up       -- to S2
Fa0/3                          up             up       -- to S1
Fa0/15                         up             up       -- to PC-C
```

Вносим изменения в конфигурацию коммутаторов:
```
S1(config)#vtp domain CCNA
S1(config)#vtp mode client
S1(config)#vtp password cisco

S3(config)#vtp domain CCNA
S3(config)#vtp mode client
S3(config)#vtp password cisco

S2(config)#vtp domain CCNA
S2(config)#vtp mode server
S2(config)#vtp password cisco
```

Состояние VTP на коммутаторах после изменений:
```
S1#sh vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0008.302d.4580
Configuration last modified by 0.0.0.0 at 0-0-00 00:00:00
Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 255
Number of existing VLANs          : 5
Configuration Revision            : 0
MD5 digest                        : 0x8B 0x58 0x3D 0x9D 0x64 0xBE 0xD5 0xF6 
                                    0x62 0xCB 0x4B 0x50 0xE5 0x9C 0x6F 0xF6 

S2#sh vtp status
VTP Version                     : 2
Configuration Revision          : 0
Maximum VLANs supported locally : 250
Number of existing VLANs        : 5
VTP Operating Mode              : Server
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x8B 0x58 0x3D 0x9D 0x64 0xBE 0xD5 0xF6 
Configuration last modified by 0.0.0.0 at 0-0-00 00:00:00
Local updater ID is 0.0.0.0 (no valid interface found)

S3#sh vtp status
VTP Version                     : 2
Configuration Revision          : 0
Maximum VLANs supported locally : 250
Number of existing VLANs        : 5
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x8B 0x58 0x3D 0x9D 0x64 0xBE 0xD5 0xF6 
Configuration last modified by 0.0.0.0 at 0-0-00 00:00:00
```

### Часть 2. Настройка DTP

#### Шаг 1. Настраиваем динамические магистральные каналы между S1 и S2.

При проверке текущего состояния каналов обнаруживается, что интерфейсы каналов на коммутаторах S1 и S3 находятся в состоянии auto, тогда как на S2 - в desirable.
Эта ситуация соответствует цели этого шага, поэтому оставляем конфигурацию интерфейсов без изменений.
Результаты проверки:
```
#sh interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       auto             802.1q         trunking      1
Fa0/3       auto             802.1q         trunking      1
Port        Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1
Fa0/3       1

S2# sh interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       desirable    802.1q         trunking      1
Fa0/3       desirable    802.1q         trunking      1
Port      Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       none
Fa0/3       1

S3#sh interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       auto         802.1q         trunking      1
Fa0/2       auto         802.1q         trunking      1
Port      Vlans allowed on trunk
Fa0/1       1-4094
Fa0/2       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/2       1
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1
Fa0/2       1
```

#### Шаг 2. Настраиваем статический магистральный канал между S1 и S3.

Вносим изменения в конфигурацию:
```
S1(config)#interface fa0/1
S1(config-if)#switchport mode trunk
S1(config-if)#end

S3(config)#interface fa0/3
S3(config-if)#switchport mode trunk
S3(config-if)#end
```

Проверяем результат изменений:
```
S1#sh interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       on               802.1q         trunking      1
Fa0/3       auto             802.1q         trunking      1
Port        Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1
Fa0/3       1

S3#sh interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       auto         802.1q         trunking      1
Fa0/3       on           802.1q         trunking      1
Port      Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1
Fa0/3       1
```

### Часть 3. Добавление сетей VLAN и назначение портов

#### Шаг 1. Добавляем сети VLAN на коммутаторах.

Вносим изменения в конфигурацию S1:
```
S2(config)#vlan 10
S2(config-vlan)#name Red
S2(config-vlan)#vlan 20
S2(config-vlan)#name Blue
S2(config-vlan)#vlan 30
S2(config-vlan)#name Yellow
S2(config-vlan)#vlan 99
S2(config-vlan)#name Management
S2(config-vlan)#end
```

Проверяем наличие добавленных vlan:
```
S2#sh vlan br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gi0/1, Gi0/2
10   Red                              active    
20   Blue                             active    
30   Yellow                           active    
99   Management                       active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

Убеждаемся, что попытка создания vlan на коммутаторе с клиентской ролью в домене VTP завершится неудачей:
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 10
VTP VLAN configuration not allowed when device is in CLIENT mode.
S1(config)#
```

#### Шаг 2. Проверяем наличие обновления VTP на коммутаторах S1 и S3.
```
S1#sh interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       on               802.1q         trunking      1
Fa0/3       auto             802.1q         trunking      1
Port        Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1,10,20,30,99
Fa0/3       1,10,20,30,99
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,20,30,99
Fa0/3       1,10,20,30,99
S1#sh vlan br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gi0/1, Gi0/2
10   Red                              active    
20   Blue                             active    
30   Yellow                           active    
99   Management                       active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 

S3#sh interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       auto         802.1q         trunking      1
Fa0/3       on           802.1q         trunking      1
Port      Vlans allowed on trunk
Fa0/1       1-4094
Fa0/3       1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1,10,20,30,99
Fa0/3       1,10,20,30,99
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,20,30,99
Fa0/3       1,10,20,30,99
S3#sh vlan br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gi0/1, Gi0/2
10   Red                              active    
20   Blue                             active    
30   Yellow                           active    
99   Management                       active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

#### Шаги 3-5. Назначение портов сетям VLAN, настройка IP-адресов на коммутаторах и проверка наличия сквозного соединения.

Изменение конфигурации:
```
S1(config)#int fa0/15
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 10
S1(config-if)#int vlan 99
S1(config-if)#ip address 192.168.99.1 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#end

S2(config)#int fa0/15
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 20
S2(config-if)#int vlan 99
S2(config-if)#ip address 192.168.99.2 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#end

S3(config)#int fa0/15
S3(config-if)#switchport mode access
S3(config-if)#switchport access vlan 10
S3(config-if)#int vlan 99
S3(config-if)#ip address 192.168.99.3 255.255.255.0
S3(config-if)#no shutdown
S3(config-if)#end
```

Проверка:
a. Эхо-запрос с компьютера PC-B на PC-A неуспешен поскольку порты Fa0/15 коммутаторов S1 и S2, к которым они присоединены находятся в различных vlan.
b. Эхо-запрос с компьютера PC-A на PC-C успешен, т.к. порты Fa0/15 коммутаторов S1 и S2, к которым они присоединены находятся в одном vlan (vid 10).
c. Эхо-запрос с коммутатора S1 на компьютер PC-A неуспешен, поскольку интерфейс управления коммутатора и порт коммутатора в сторону ПК находятся в различных vlan.
d. Эхо-запрос с коммутатора S2 на коммутатор S1 был успешен, поскольку интерфейсы управления коммутаторов принадлежат одному vlan (vid 99).

### Часть 4. Настройка расширенной сети VLAN

#### Шаг 1. Переводим VTP на коммутаторе S1 в прозрачный режим и проверяем результат.

```
S1#conf t
S1(config)#vtp mode transparent
S1(config)#end

S1#sh vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0008.302d.4580
Configuration last modified by 0.0.0.0 at 3-1-93 00:34:07
Feature VLAN:
--------------
VTP Operating Mode                : Transparent
Maximum VLANs supported locally   : 255
Number of existing VLANs          : 9
Configuration Revision            : 0
MD5 digest                        : 0xB2 0x9A 0x11 0x5B 0xBF 0x2E 0xBF 0xAA 
                                    0x31 0x18 0xFF 0x2C 0x5E 0x54 0x0A 0xB7 
```

#### Шаг 2. Настраиваем сеть VLAN расширенного диапазона на коммутаторе S1.

```
S1#sh vlan br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gi0/1, Gi0/2
10   Red                              active    Fa0/15
20   Blue                             active    
30   Yellow                           active    
99   Management                       active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 

S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 2000
S1(config-vlan)#end

S1#sh vlan br
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gi0/1, Gi0/2
10   Red                              active    Fa0/15
20   Blue                             active    
30   Yellow                           active    
99   Management                       active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
2000 VLAN2000                         active    
```

Таким образом, убеждаемся в возможности управления собственной базой vlan на коммутаторе и одновременной работе по передаче объявлений VTP.

