# Настройка базового протокола OSPFv2 для одной области

###  Задачи:

  1. Создание сети и настройка основных параметров устройства
  2. Настройка и проверка маршрутизации OSPF
  3. Изменение назначений идентификаторов маршрутизаторов
  4. Настройка пассивных интерфейсов OSPF
  5. Изменение метрик OSPF

### Топология:
![](images/otus-ospf.png)

### Таблица адресации:

| Устройство | Интерфейс     | IP-адрес     | Маска подсети   | Шлюз по умолчанию |
|:-----------|:--------------|:-------------|:----------------|:-----------------:|
| R1         | Gi0/0         | 192.168.1.1  | 255.255.255.0   | -                 |
|            | Se0/0/1       | 192.168.12.1 | 255.255.255.252 | -                 |
|            | Se0/0/0       | 192.168.13.1 | 255.255.255.252 | -                 |
| R2         | Fa0/0         | 192.168.2.1  | 255.255.255.0   | -                 |
|            | Se0/1/1 (DCE) | 192.168.12.2 | 255.255.255.252 | -                 |
|            | Se0/1/0       | 192.168.23.1 | 255.255.255.252 | -                 |
| R3         | Fa0/0         | 192.168.3.1  | 255.255.255.0   | -                 |
|            | Se0/3/0 (DCE) | 192.168.13.2 | 255.255.255.252 | -                 |
|            | Se0/3/1 (DCE) | 192.168.23.2 | 255.255.255.252 | -                 |
| PC-A       | NIC           | 192.168.1.3  | 255.255.255.0   | 192.168.1.1       |
| PC-B       | NIC           | 192.168.2.3  | 255.255.255.0   | 192.168.2.1       |
| PC-C       | NIC           | 192.168.3.3  | 255.255.255.0   | 192.168.3.1       |

## Часть 1. Создание сети и настройка основных параметров устройства

Файл изменений конфигурации маршрутизатора R1: [R1.conf](configs/R1_conf_part1.txt)  
Файл изменений конфигурации маршрутизатора R2: [R2.conf](configs/R2_conf_part1.txt)  
Файл изменений конфигурации маршрутизатора R3: [R3.conf](configs/R3_conf_part1.txt)

Результат изменений:

```
R1#sh int descr | i up
Interface                      Status         Protocol Description
Gi0/0                          up             up       -- to PC-A
Se0/0/0                        up             up       -- to R3
Se0/0/1                        up             up       -- to R2
R1#sh ip int br | i up
GigabitEthernet0/0         192.168.1.1     YES manual up                    up      
Serial0/0/0                192.168.13.1    YES manual up                    up      
Serial0/0/1                192.168.12.1    YES manual up                    up      
```

```
R2#sh int descr | i up
Interface                      Status         Protocol Description
Fa0/0                          up             up       -- to PC-B
Se0/1/0                        up             up       -- to R3
Se0/1/1                        up             up       -- to R2
R2#sh ip int br | i up
FastEthernet0/0            192.168.2.1     YES manual up                    up      
Serial0/1/0                192.168.23.1    YES manual up                    up      
Serial0/1/1                192.168.12.2    YES manual up                    up      
R2#sh controller s0/1/1 | i clock
DCE V.35, clock rate 128000
R2#ping 192.168.12.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.12.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/15/16 ms
```

```
R3#sh int descr | i up
Interface                      Status         Protocol Description
Fa0/0                          up             up       -- to PC-C
Se0/3/0                        up             up       -- to R1
Se0/3/1                        up             up       -- to R2
R3#sh ip int br | i up
FastEthernet0/0            192.168.3.1     YES manual up                    up      
Serial0/3/0                192.168.13.2    YES SLARP  up                    up      
Serial0/3/1                192.168.23.2    YES SLARP  up                    up      
R3#sh runn | s banner
banner motd ^C
R3#sh controller s0/3/0 | i clock
DCE V.35, clock rate 128000
R3#sh controller s0/3/1 | i clock
DCE V.35, clock rate 128000
R3#
R3#ping 192.168.13.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.13.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/14/16 ms
R3#ping 192.168.23.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/14/16 ms
```

## Часть 2. Настройка и проверка маршрутизации OSPF

Файл изменений конфигурации маршрутизатора R1: [R1 config](configs/R1_conf_part2.txt)  
Файл изменений конфигурации маршрутизатора R1: [R2 config](configs/R2_conf_part2.txt)  
Файл изменений конфигурации маршрутизатора R1: [R3 config](configs/R3_conf_part2.txt)

#### Результаты:  
##### Шаг 3. Проверка информации о соседних устройствах и маршрутизации OSPF.

```
R1#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.23.2      0   FULL/  -        00:00:38    192.168.13.2    Serial0/0/0
192.168.23.1      0   FULL/  -        00:00:33    192.168.12.2    Serial0/0/1

R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/782] via 192.168.12.2, 00:02:28, Serial0/0/1
O     192.168.3.0/24 [110/782] via 192.168.13.2, 00:01:44, Serial0/0/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/845] via 192.168.13.2, 00:01:44, Serial0/0/0
```

##### Шаг 4. Проверка параметров протокола OSPF.

```
R1#sh ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 192.168.13.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.23.2         110      00:02:19
    192.168.23.1         110      00:03:03
  Distance: (default is 110)
```

##### Шаг 5. Проверка данных процесса OSPF.

```
R1#sh ip ospf
 Routing Process "ospf 1" with ID 192.168.13.1
 Start time: 00:46:40.796, Time elapsed: 00:04:16.120
 Supports only single TOS(TOS0) routes
 Supports opaque LSA
 Supports Link-local Signaling (LLS)
 Supports area transit capability
 Event-log enabled, Maximum number of events: 1000, Mode: cyclic
 Router is not originating router-LSAs with maximum metric
 Initial SPF schedule delay 5000 msecs
 Minimum hold time between two consecutive SPFs 10000 msecs
 Maximum wait time between two consecutive SPFs 10000 msecs
 Incremental-SPF disabled
 Minimum LSA interval 5 secs
 Minimum LSA arrival 1000 msecs
 LSA group pacing timer 240 secs
 Interface flood pacing timer 33 msecs
 Retransmission pacing timer 66 msecs
 Number of external LSA 0. Checksum Sum 0x000000
 Number of opaque AS LSA 0. Checksum Sum 0x000000
 Number of DCbitless external and opaque AS LSA 0
 Number of DoNotAge external and opaque AS LSA 0
 Number of areas in this router is 1. 1 normal 0 stub 0 nssa
 Number of areas transit capable is 0
 External flood list length 0
 IETF NSF helper support enabled
 Cisco NSF helper support enabled
 Reference bandwidth unit is 100 mbps
    Area BACKBONE(0)
        Number of interfaces in this area is 3
Area has no authentication
SPF algorithm last executed 00:02:23.904 ago
SPF algorithm executed 3 times
Area ranges are
Number of LSA 3. Checksum Sum 0x01582B
Number of opaque link LSA 0. Checksum Sum 0x000000
Number of DCbitless LSA 0
Number of indication LSA 0
Number of DoNotAge LSA 0
Flood list length 0
```

##### Шаг 6. Проверка параметров интерфейса OSPF.

```
R1#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/0/0      1     0               192.168.13.1/30    781   P2P   1/1
Se0/0/1      1     0               192.168.12.1/30    781   P2P   1/1
Gi0/0        1     0               192.168.1.1/24     1     DR    0/0
R1#

R1#show ip ospf interface      
Serial0/0/0 is up, line protocol is up 
  Internet Address 192.168.13.1/30, Area 0 
  Process ID 1, Router ID 192.168.13.1, Network Type POINT_TO_POINT, Cost: 781
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           781       no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:00
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 3/3, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 192.168.23.2
  Suppress hello for 0 neighbor(s)
Serial0/0/1 is up, line protocol is up 
  Internet Address 192.168.12.1/30, Area 0 
  Process ID 1, Router ID 192.168.13.1, Network Type POINT_TO_POINT, Cost: 781
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           781       no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:00
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 2/2, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 192.168.23.1
  Suppress hello for 0 neighbor(s)
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 192.168.1.1/24, Area 0 
  Process ID 1, Router ID 192.168.13.1, Network Type BROADCAST, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 192.168.13.1, Interface address 192.168.1.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:09
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0 
  Suppress hello for 0 neighbor(s)
```

##### Шаг 7. Проверка наличия сквозного соединения.

Эхо-запросы с PC-A до PC-B, PC-C прошли успешно.


## Часть 3. Изменение назначений идентификаторов маршрутизаторов


##### Шаг 2. Изменение идентификаторов маршрутизатора с помощью loopback-адресов.

Результат успешный - после перезагрузки роутер id для ospf изменился на заданный в lo0 интерфейсе.

##### Шаг 3. Изменение идентификатора маршрутизатора с помощью команды router-id.

Вносим измнения в конфиг маршрутизаторов и перезагружаем ospf:

**R1:**
```
conf t
router ospf 1
router-id 11.11.11.11
end
clear ip ospf process
```

**R2:**
```
conf t
router ospf 1
router-id 22.22.22.22
end
clear ip ospf process
```

**R3:**
```
conf t
router ospf 1
router-id 33.33.33.33
end
clear ip ospf process
```

Результат:
```
R1# sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:38    192.168.13.2    Serial0/0/0
22.22.22.22       0   FULL/  -        00:00:35    192.168.12.2    Serial0/0/1

R2#sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:31    192.168.23.2    Serial0/1/0
11.11.11.11       0   FULL/  -        00:00:38    192.168.12.1    Serial0/1/1

R3#sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
22.22.22.22       0   FULL/  -        00:00:31    192.168.23.1    Serial0/3/1
11.11.11.11       0   FULL/  -        00:00:32    192.168.13.1    Serial0/3/0
```

## Часть 4. Настройка пассивных интерфейсов OSPF

##### Шаг 1. Настройка пассивного интерфейса.

Смотрим текущее состояние интерфейса:
```
R1#sh ip ospf int gi0/0
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 192.168.1.1/24, Area 0 
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 11.11.11.11, Interface address 192.168.1.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:08
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0 
  Suppress hello for 0 neighbor(s)
```

Вносим изменения в конфигурацию маршрутизатора:
```
router ospf 1
passive-interface gi0/0
end
```

Наблюдаем изменения в состоянии интерфейса:
```
R1#sh ip ospf int gi0/0
*Jun  6 07:28:06.071: %SYS-5-CONFIG_I: Configured from console by console
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 192.168.1.1/24, Area 0 
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Transmit Delay is 1 sec, State WAITING, Priority 1
  No designated router on this network
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface) 
    Wait time before Designated router selection 00:00:32
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0 
  Suppress hello for 0 neighbor(s)
```

Проверяем наличие маршрутов на соседних коммутаторах:
```
R2#sh ip route ospf
     192.168.13.0/30 is subnetted, 1 subnets
O       192.168.13.0 [110/845] via 192.168.23.2, 00:03:16, Serial0/1/0
O    192.168.1.0/24 [110/782] via 192.168.12.1, 00:03:39, Serial0/1/1
O    192.168.3.0/24 [110/782] via 192.168.23.2, 00:03:16, Serial0/1/0

R3#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.1.0/24 [110/65] via 192.168.13.1, 00:04:44, Serial0/3/0
O     192.168.2.0/24 [110/65] via 192.168.23.1, 00:04:44, Serial0/3/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/845] via 192.168.23.1, 00:04:44, Serial0/3/1
                      [110/845] via 192.168.13.1, 00:04:44, Serial0/3/0
```

##### Шаг 2. Настройка на маршрутизаторе пассивного интерфейса в качестве интерфейса по умолчанию.

Фиксируем текущее состояние ospf-соседства:
```
R1#sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:37    192.168.13.2    Serial0/0/0
22.22.22.22       0   FULL/  -        00:00:32    192.168.12.2    Serial0/0/1
```

Вносим изменения в R2:
```
R2(config)#router ospf 1
R2(config-router)#passive-interface default
R2(config-router)#end
```

Проверяем изменения на R1:
```
R1#
*Jun  6 07:34:07.879: %OSPF-5-ADJCHG: Process 1, Nbr 22.22.22.22 on Serial0/0/1 from FULL to DOWN, Neighbor Down: Dead timer expired
R1#sh ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:39    192.168.13.2    Serial0/0/0
```

Проверяем состояние OSPF для интерфейса S0/1/1 маршрутизатора R2:
```
R2#show ip ospf interface s0/1/1
Serial0/1/1 is up, line protocol is up 
  Internet Address 192.168.12.2/30, Area 0 
  Process ID 1, Router ID 22.22.22.22, Network Type POINT_TO_POINT, Cost: 781
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface) 
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 2/2, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0 
  Suppress hello for 0 neighbor(s)
```

Разрешаем OSPF анонсировать маршруты для линковочных интерфейсов между маршрутизаторами:

```
R2(config)#router ospf 1
R2(config-router)#no passive-interface se0/1/1
R2(config-router)#no passive-interface se0/1/0
R2(config-router)#end

R1(config)#router ospf 1
R1(config-router)#no passive-interface se0/0/0
R1(config-router)#no passive-interface se0/0/1
R1(config-router)#end

R3(config)#router ospf 1
R3(config-router)#no passive-interface se0/3/0
R3(config-router)#no passive-interface se0/3/1
R3(config-router)#end
```

Проверяем корректность настроек:
```
R1#sh ip protocols 
*** IP Routing is NSF aware ***
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 11.11.11.11
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Passive Interface(s):
    GigabitEthernet0/0
    GigabitEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
    33.33.33.33          110      00:02:16
    22.22.22.22          110      00:05:11
    192.168.23.2         110      00:06:14
    192.168.23.1         110      00:06:04
  Distance: (default is 110)
R1#sh ip ospf int br
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/0/1      1     0               192.168.12.1/30    781   P2P   1/1
Se0/0/0      1     0               192.168.13.1/30    781   P2P   1/1
Gi0/0        1     0               192.168.1.1/24     1     DR    0/0
R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/782] via 192.168.12.2, 00:05:41, Serial0/0/1
O     192.168.3.0/24 [110/782] via 192.168.13.2, 00:02:46, Serial0/0/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/845] via 192.168.13.2, 00:02:46, Serial0/0/0
```

```
R2#sh ip protocols 
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 22.22.22.22
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.2.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.23.0 0.0.0.3 area 0
 Reference bandwidth unit is 100 mbps
  Passive Interface(s):
    FastEthernet0/0
    FastEthernet0/1
    VoIP-Null0
  Routing Information Sources:
    Gateway         Distance      Last Update
    33.33.33.33          110      00:04:09
    11.11.11.11          110      00:05:42
    192.168.23.2         110      00:05:32
  Distance: (default is 110)
R2#sh ip ospf int br
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/1/0      1     0               192.168.23.1/30    781   P2P   1/1
Se0/1/1      1     0               192.168.12.2/30    781   P2P   1/1
Fa0/0        1     0               192.168.2.1/24     1     DR    0/0
R2#sh ip route ospf
     192.168.13.0/30 is subnetted, 1 subnets
O       192.168.13.0 [110/845] via 192.168.23.2, 00:04:27, Serial0/1/0
O    192.168.1.0/24 [110/782] via 192.168.12.1, 00:07:22, Serial0/1/1
O    192.168.3.0/24 [110/782] via 192.168.23.2, 00:04:27, Serial0/1/0
```

```
R3#sh ip protocols 
*** IP Routing is NSF aware ***
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 33.33.33.33
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.3.0 0.0.0.255 area 0
    192.168.13.0 0.0.0.3 area 0
    192.168.23.0 0.0.0.3 area 0
  Passive Interface(s):
    FastEthernet0/0
    FastEthernet0/1
    VoIP-Null0
  Routing Information Sources:
    Gateway         Distance      Last Update
    22.22.22.22          110      00:05:01
    11.11.11.11          110      00:05:01
    192.168.13.1         110      00:11:27
    192.168.23.1         110      00:08:12
  Distance: (default is 110)
          
R3#sh ip ospf int br
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/3/1      1     0               192.168.23.2/30    64    P2P   1/1
Se0/3/0      1     0               192.168.13.2/30    64    P2P   1/1
Fa0/0        1     0               192.168.3.1/24     1     DR    0/0
R3#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.1.0/24 [110/65] via 192.168.13.1, 00:05:17, Serial0/3/0
O     192.168.2.0/24 [110/65] via 192.168.23.1, 00:05:17, Serial0/3/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/845] via 192.168.23.1, 00:05:17, Serial0/3/1
                      [110/845] via 192.168.13.1, 00:05:17, Serial0/3/0
```

## Часть 5. Изменение метрик OSPF

##### Шаг 1. Изменяем заданную пропускную способность для маршрутизаторов.

Проверяем текущее значение пропускной способности интерфейса gi0/0 на R1:
```
R1#sh int gi0/0
GigabitEthernet0/0 is up, line protocol is up 
  Hardware is CN Gigabit Ethernet, address is e8b7.484a.1100 (bia e8b7.484a.1100)
  Description: -- to PC-A
  Internet address is 192.168.1.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, 
...
```

Смотрим чему равна суммарная метрика для маршрута к сети 192.168.3.0/24:
```
R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/782] via 192.168.12.2, 00:29:52, Serial0/0/1
O     192.168.3.0/24 [110/782] via 192.168.13.2, 00:06:28, Serial0/0/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/845] via 192.168.13.2, 00:02:45, Serial0/0/0
```

Как следует из вывода команды, суммарная метрика равна 782 и равна сумме стоимости интерфейса fa0/0 на маршрутизаторе R3 и стоимости интерфейса s0/0/0 на R1:
```
R3#sh ip ospf int fa0/0    
FastEthernet0/0 is up, line protocol is up 
  Internet Address 192.168.3.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 33.33.33.33, Network Type BROADCAST, Cost: 1
...

R1#sh ip ospf int se0/0/0
Serial0/0/0 is up, line protocol is up 
  Internet Address 192.168.13.1/30, Area 0 
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 781
...
```


Изменим параметр эталонной пропускной способности по умолчанию на всех маршрутизаторах:
```
R1(config)#router ospf 1
R1(config-router)#auto-cost reference-bandwidth 10000
R1(config-router)#end

R2(config)#router ospf 1
R2(config-router)#auto-cost reference-bandwidth 10000
R2(config-router)#end

R3(config)#router ospf 1
R3(config-router)#auto-cost reference-bandwidth 10000
R3(config-router)#end
```

И проверим изменение суммарной стоимости для маршрутов:
```
R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/65635] via 192.168.12.2, 00:05:02, Serial0/0/1
O     192.168.3.0/24 [110/65635] via 192.168.13.2, 00:04:49, Serial0/0/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/72011] via 192.168.13.2, 00:04:49, Serial0/0/0


R2#sh ip route ospf
     192.168.13.0/30 is subnetted, 1 subnets
O       192.168.13.0 [110/72011] via 192.168.23.2, 00:05:14, Serial0/1/0
O    192.168.1.0/24 [110/65545] via 192.168.12.1, 00:05:24, Serial0/1/1
O    192.168.3.0/24 [110/65635] via 192.168.23.2, 00:05:14, Serial0/1/0


R3#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.1.0/24 [110/6486] via 192.168.13.1, 00:05:43, Serial0/3/0
O     192.168.2.0/24 [110/6576] via 192.168.23.1, 00:05:43, Serial0/3/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/72011] via 192.168.23.1, 00:05:43, Serial0/3/1
                      [110/72011] via 192.168.13.1, 00:05:43, Serial0/3/0
```

Суммарная стоимость маршрута для 192.168.3.0/24 на R1 теперь равна 65635 = 65535 + 100

```
R1#sh ip ospf int se0/0/0 | i Cost:
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 65535

R3#sh ip ospf int fa0/0 | i Cost:
  Process ID 1, Router ID 33.33.33.33, Network Type BROADCAST, Cost: 100
```

(Возвращаем значение эталонной пропускной способности OSPF командой `auto-cost reference-bandwidth 100`).

##### Шаг 2. Изменение пропускной способности для интерфейса.

Проверим связь маршрутов с пропускной способностью каналов.  

Вначале посмотрим текущее состояние интерфейса S0/3/1 и маршрутной информации OSPF на R3:
```
R3#sh int se0/3/1              
Serial0/3/1 is up, line protocol is up 
  Hardware is GT96K Serial
  Description: -- to R2
  Internet address is 192.168.23.2/30
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec, 

R3#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.1.0/24 [110/65] via 192.168.13.1, 00:15:23, Serial0/3/0
O     192.168.2.0/24 [110/65] via 192.168.23.1, 00:00:07, Serial0/3/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/845] via 192.168.23.1, 00:00:07, Serial0/3/1
                      [110/845] via 192.168.13.1, 00:15:23, Serial0/3/0

R3#sh ip ospf int br           
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/3/1      1     0               192.168.23.2/30    64    P2P   1/1
Se0/3/0      1     0               192.168.13.2/30    64    P2P   1/1
Fa0/0        1     0               192.168.3.1/24     1     DR    0/0
```

Задаем значение пропускной способности:
```
R3(config)#int se0/3/1
R3(config-if)#bandwidth 128
R3(config-if)#end
```

И проверяем состояние интерфейса и маршруты:
```
R3#sh int se0/3/1
Serial0/3/1 is up, line protocol is up 
  Hardware is GT96K Serial
  Description: -- to R2
  Internet address is 192.168.23.2/30
  MTU 1500 bytes, BW 128 Kbit/sec, DLY 20000 usec,

R3#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.1.0/24 [110/65] via 192.168.13.1, 00:14:54, Serial0/3/0
O     192.168.2.0/24 [110/782] via 192.168.23.1, 00:07:35, Serial0/3/1
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/845] via 192.168.13.1, 00:14:54, Serial0/3/0

R3#sh ip ospf int br           
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se0/3/1      1     0               192.168.23.2/30    781   P2P   1/1
Se0/3/0      1     0               192.168.13.2/30    64    P2P   1/1
Fa0/0        1     0               192.168.3.1/24     1     DR    0/0
```

Видим, что из таблицы маршрутизации исчез маршрут для 192.168.23.0/30 через S0/3/1 в связи возросшей с 64 до 781 стоимостью интерфейса.

##### Шаг 3. Изменение стоимости маршрута.

Зафиксируем текущее состояние OSPF:
```
R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/782] via 192.168.12.2, 00:05:40, Serial0/0/1
O     192.168.3.0/24 [110/782] via 192.168.13.2, 00:06:30, Serial0/0/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/1562] via 192.168.13.2, 00:02:12, Serial0/0/0
                      [110/1562] via 192.168.12.2, 00:02:12, Serial0/0/1
```

Изменяем стоимость канала на R1:
```
R1(config)#int se0/0/0
R1(config-if)#ip ospf cost 1565
R1(config-if)#end
```

И проверяем изменение в состоянии OSPF:
```
R1#sh ip route ospf | b Gateway
Gateway of last resort is not set
O     192.168.2.0/24 [110/782] via 192.168.12.2, 00:20:26, Serial0/0/1
O     192.168.3.0/24 [110/1563] via 192.168.12.2, 00:07:33, Serial0/0/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/1562] via 192.168.12.2, 00:16:58, Serial0/0/1
```

Видим, что после увеличения стоимости канала в сторону R3 все маршруты теперь проходят через R2.

