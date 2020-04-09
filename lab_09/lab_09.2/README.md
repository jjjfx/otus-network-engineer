# Настройка расширенных функций EIGRP для IPv4

### Топология:
![](images/otus-topology.png)

### Таблица адресации:

| Устройство | Интерфейс     | IP-адрес     | Маска подсети   | Шлюз по умолчанию |
|:-----------|:--------------|:-------------|:----------------|:-----------------:|
| R1         | Gi0/0         | 192.168.1.1  | 255.255.255.0   | -                 |
|            | Se0/0/1       | 192.168.12.1 | 255.255.255.252 | -                 |
|            | Se0/0/0       | 192.168.13.1 | 255.255.255.252 | -                 |
|            | Lo1           | 192.168.11.1 | 255.255.255.252 | -                 |
|            | Lo5           | 192.168.11.5 | 255.255.255.252 | -                 |
|            | Lo9           | 192.168.11.9 | 255.255.255.252 | -                 |
|            | Lo13          | 192.168.11.13| 255.255.255.252 | -                 |
| R2         | Fa0/0         | 192.168.2.1  | 255.255.255.0   | -                 |
|            | Se0/1/1 (DCE) | 192.168.12.2 | 255.255.255.252 | -                 |
|            | Se0/1/0       | 192.168.23.1 | 255.255.255.252 | -                 |
|            | Lo1           | 192.168.22.1 | 255.255.255.252 | -                 |
| R3         | Fa0/0         | 192.168.3.1  | 255.255.255.0   | -                 |
|            | Se0/3/0 (DCE) | 192.168.13.2 | 255.255.255.252 | -                 |
|            | Se0/3/1 (DCE) | 192.168.23.2 | 255.255.255.252 | -                 |
|            | Lo1           | 192.168.33.1 | 255.255.255.252 | -                 |
|            | Lo5           | 192.168.33.5 | 255.255.255.252 | -                 |
|            | Lo9           | 192.168.33.9 | 255.255.255.252 | -                 |
|            | Lo13          | 192.168.33.13| 255.255.255.252 | -                 |
| PC-A       | NIC           | 192.168.1.3  | 255.255.255.0   | 192.168.1.1       |
| PC-B       | NIC           | 192.168.2.3  | 255.255.255.0   | 192.168.2.1       |
| PC-C       | NIC           | 192.168.3.3  | 255.255.255.0   | 192.168.3.1       |

## Часть 1. Создание сети и настройка основных параметров устройства

Файл изменений конфигурации маршрутизатора R1: [R1.conf](configs/R1_conf.txt)  
Файл изменений конфигурации маршрутизатора R2: [R2.conf](configs/R2_conf.txt)  
Файл изменений конфигурации маршрутизатора R3: [R3.conf](configs/R3_conf.txt)


R1(config)#router eigrp 1
R1(config-router)#network 192.168.1.0 255.255.255.0
R1(config-router)#network 192.168.12.0 255.255.255.252
R1(config-router)#network 192.168.13.0 255.255.255.252
R1(config-router)#exit
R1(config)#int gi0/0
R1(config-if)#eig
R1(config-if)#paa
R1(config-if)#pass
R1(config-if)#no ip hell
R1(config-if)#no ip hello-interval ?
  eigrp  Enhanced Interior Gateway Routing Protocol (EIGRP)
R1(config-if)#no ip hello-interval eigrp
% Incomplete command.
R1(config-if)#no ip hello-interval eigrp ?
  <1-65535>  AS number
R1(config-if)#no ip hello-interval eigrp 1
R1(config-if)#


R2(config)#router eigrp 1
R2(config-router)#network 192.168.2.0 255.255.255.0
R2(config-router)#network 192.168.12.2 255.255.255.252
R2(config-router)#network 192.168.12.2 255.255.255.252
*Jan  1 00:36:42.851: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.1 (Serial0/1/1) is up: new adjacency
R2(config-router)#network 192.168.23.0 255.255.255.252
R2(config-router)#exit
R2(config)#int fa0/0
R2(config-if)#no ip hell
R2(config-if)#no ip hello-interval eig
R2(config-if)#no ip hello-interval eigrp 1
R2(config-if)#int se0/1/1
R2(config-if)#band
R2(config-if)#bandwidth 1024
R2(config-if)#end


R3(config)#router eigh
R3(config)#router eigr
R3(config)#router eigrp 1
R3(config-router)#network 192.168.3.0 255.255.255.0
R3(config-router)#network 192.168.23.2 255.255.255.252
R3(config-router)#network 192.168.23.2 255.255.255.252
Jan  2 12:26:39.747: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.23.1 (Serial0/3/1) is up: new adjacency
R3(config-router)#network 192.168.13.2 255.255.255.252
R3(config-router)#
Jan  2 12:26:44.783: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.1 (Serial0/3/0) is up: new adjacency
R3(config-router)#exit
R3(config)#int fa0/0
R3(config-if)#no ip hell
R3(config-if)#no ip hello-interval eigrp 1
R3(config-if)#int se0/3/0
R3(config-if)#bandwidth 64
R3(config-if)#end


Пинг успешный - двигаемся дальше.

R1#sh ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  Redistributing: eigrp 1
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    NSF-aware route hold timer is 240
    Router-ID: 192.168.13.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    192.168.1.0
    192.168.12.0/30
    192.168.13.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.12.2          90      00:01:39
    192.168.13.2          90      00:01:40
  Distance: internal 90 external 170
R1#

R2#sh ip protocols
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  EIGRP maximum hopcount 100
  EIGRP maximum metric variance 1
  Redistributing: eigrp 1
  EIGRP NSF-aware route hold timer is 240s
  Automatic network summarization is in effect
  Automatic address summarization:
    192.168.23.0/24 for FastEthernet0/0, Serial0/1/1
      Summarizing with metric 20512000
    192.168.12.0/24 for FastEthernet0/0, Serial0/1/0
      Summarizing with metric 3011840
    192.168.2.0/24 for Serial0/1/1, Serial0/1/0
  Maximum path: 4
  Routing for Networks:
    192.168.2.0
    192.168.12.0/30
    192.168.23.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    (this router)         90      00:02:24
    192.168.12.1          90      00:02:24
    192.168.23.2          90      00:02:24
  Distance: internal 90 external 170


R3#sh ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    NSF-aware route hold timer is 240
    Router-ID: 192.168.23.2
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    192.168.3.0
    192.168.13.0/30
    192.168.23.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.13.1          90      00:02:39
    192.168.23.1          90      00:02:39
  Distance: internal 90 external 170
R3#

R1#sh ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  Redistributing: eigrp 1
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    NSF-aware route hold timer is 240
    Router-ID: 192.168.13.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    192.168.1.0
    192.168.12.0/30
    192.168.13.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.12.2          90      00:01:39
    192.168.13.2          90      00:01:40
  Distance: internal 90 external 170
R1#sh ip eigrp nei
EIGRP-IPv4 Neighbors for AS(1)
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
1   192.168.13.2            Se0/0/0           10 00:03:50   13  2280  0  16
0   192.168.12.2            Se0/0/1           13 00:05:24   10   200  0  17
R1#sh run | s router
router eigrp 1
 network 192.168.1.0
 network 192.168.12.0 0.0.0.3
 network 192.168.13.0 0.0.0.3
R1#sh run int gi0/0
Building configuration...
Current configuration : 123 bytes
!
interface GigabitEthernet0/0
 description -- to PC-A
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
end


R2#sh ip protocols
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  EIGRP maximum hopcount 100
  EIGRP maximum metric variance 1
  Redistributing: eigrp 1
  EIGRP NSF-aware route hold timer is 240s
  Automatic network summarization is in effect
  Automatic address summarization:
    192.168.23.0/24 for FastEthernet0/0, Serial0/1/1
      Summarizing with metric 20512000
    192.168.12.0/24 for FastEthernet0/0, Serial0/1/0
      Summarizing with metric 3011840
    192.168.2.0/24 for Serial0/1/1, Serial0/1/0
  Maximum path: 4
  Routing for Networks:
    192.168.2.0
    192.168.12.0/30
    192.168.23.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    (this router)         90      00:02:24
    192.168.12.1          90      00:02:24
    192.168.23.2          90      00:02:24
  Distance: internal 90 external 170
R2#sh ip eigrp nei
IP-EIGRP neighbors for process 1
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
1   192.168.23.2            Se0/1/0           12 00:03:51    9  1140  0  17
0   192.168.12.1            Se0/1/1           14 00:05:20    7   200  0  11
R2#sh run | s router
router eigrp 1
 network 192.168.2.0
 network 192.168.12.0 0.0.0.3
 network 192.168.23.0 0.0.0.3
 auto-summary
R2#sh run int fa0/0
Building configuration...
Current configuration : 120 bytes
!
interface FastEthernet0/0
 description -- to PC-B
 ip address 192.168.2.1 255.255.255.0
 duplex auto
 speed auto
end


R3#sh ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    NSF-aware route hold timer is 240
    Router-ID: 192.168.23.2
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    192.168.3.0
    192.168.13.0/30
    192.168.23.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.13.1          90      00:02:39
    192.168.23.1          90      00:02:39
  Distance: internal 90 external 170
R3#
R3#sh ip eigrp nei
EIGRP-IPv4 Neighbors for AS(1)
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
1   192.168.13.1            Se0/3/0           10 00:03:32   19  2280  0  10
0   192.168.23.1            Se0/3/1           10 00:03:37   15   200  0  18
R3#sh run | s router
router eigrp 1
 network 192.168.3.0
 network 192.168.13.0 0.0.0.3
 network 192.168.23.0 0.0.0.3


R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#router eigrp 1
R1(config-router)#auto-summary
R1(config-router)#
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.2 (Serial0/0/0) is resync: summary configured
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is resync: summary configured
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.2 (Serial0/0/0) is resync: summary up, remove components
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is resync: summary up, remove components
R1(config-router)#
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is resync: summary up, remove components
*Jun 18 11:45:43.171: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.2 (Serial0/0/0) is resync: summary up, remove components


R2#sh ip route eigrp
     192.168.13.0/30 is subnetted, 1 subnets
D       192.168.13.0 [90/41024000] via 192.168.23.2, 00:21:07, Serial0/1/0
                     [90/41024000] via 192.168.12.1, 00:21:07, Serial0/1/1
     192.168.11.0/30 is subnetted, 4 subnets
D       192.168.11.0 [90/3139840] via 192.168.12.1, 00:10:18, Serial0/1/1
D       192.168.11.4 [90/3139840] via 192.168.12.1, 00:10:18, Serial0/1/1
D       192.168.11.8 [90/3139840] via 192.168.12.1, 00:10:18, Serial0/1/1
D       192.168.11.12 [90/3139840] via 192.168.12.1, 00:10:18, Serial0/1/1
D    192.168.1.0/24 [90/3012096] via 192.168.12.1, 00:21:07, Serial0/1/1
D    192.168.3.0/24 [90/20514560] via 192.168.23.2, 00:21:47, Serial0/1/0
R2#
R2#
*Jan  1 01:00:13.871: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.1 (Serial0/1/1) is resync: peer graceful-restart
R2#sh ip route eigrp
     192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
D       192.168.12.0/24 [90/41536000] via 192.168.23.2, 00:00:10, Serial0/1/0
     192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
D       192.168.13.0/30 [90/41024000] via 192.168.23.2, 00:00:10, Serial0/1/0
D       192.168.13.0/24 [90/41024000] via 192.168.12.1, 00:00:10, Serial0/1/1
D    192.168.11.0/24 [90/3139840] via 192.168.12.1, 00:00:10, Serial0/1/1
D    192.168.1.0/24 [90/3012096] via 192.168.12.1, 00:21:31, Serial0/1/1
D    192.168.3.0/24 [90/20514560] via 192.168.23.2, 00:22:12, Serial0/1/0
R2#

R3(config-router)#router eigrp 1
R3(config-router)#network 192.168.33.0 255.255.255.252
R3(config-router)#network 192.168.33.4 255.255.255.252
R3(config-router)#network 192.168.33.8 255.255.255.252
R3(config-router)#network 192.168.33.12 255.255.255.252
R3(config-router)#auto-summ
R3(config-router)#auto-summary 
R3(config-router)#
Jan  2 12:50:47.947: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.1 (Serial0/3/0) is resync: summary configured
Jan  2 12:50:47.947: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.23.1 (Serial0/3/1) is resync: summary configured
Jan  2 12:50:47.951: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.1 (Serial0/3/0) is resync: summary up, remove components
Jan  2 12:50:47.951: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.23.1 (Serial0/3/1) is resync: summary up, remove components
Jan  2 12:50:47.955: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.23.1 (Serial0/3/1) is resync: summary up, remove components
R3(config-router)#
Jan  2 12:50:47.955: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.13.1 (Serial0/3/0) is resync: summary up, remove components
R3(config-router)#end
R3#


R2#sh ip route eigrp
     192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
D       192.168.12.0/24 [90/41536000] via 192.168.23.2, 00:02:15, Serial0/1/0
D    192.168.13.0/24 [90/41024000] via 192.168.23.2, 00:00:08, Serial0/1/0
                     [90/41024000] via 192.168.12.1, 00:00:08, Serial0/1/1
D    192.168.11.0/24 [90/3139840] via 192.168.12.1, 00:02:15, Serial0/1/1
     192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
D       192.168.23.0/24 [90/41536000] via 192.168.12.1, 00:00:08, Serial0/1/1
D    192.168.1.0/24 [90/3012096] via 192.168.12.1, 00:23:36, Serial0/1/1
D    192.168.3.0/24 [90/20514560] via 192.168.23.2, 00:24:17, Serial0/1/0
     192.168.33.0/24 is subnetted, 1 subnets
D       192.168.33.0 [90/20640000] via 192.168.23.2, 00:00:08, Serial0/1/0



Настройка и распространение статического маршрута по умолчанию

R2(config)#ip route 0.0.0.0 0.0.0.0 Lo1
R2(config)#router eigrp 1
R2(config-router)#redistribute static
R2(config-router)#end
R2#
*Jan  1 01:05:21.491: %SYS-5-CONFIG_I: Configured from console by console
R2#sh ip protocols
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  EIGRP maximum hopcount 100
  EIGRP maximum metric variance 1
  Redistributing: static, eigrp 1
  EIGRP NSF-aware route hold timer is 240s
  Automatic network summarization is not in effect
  Maximum path: 4
  Routing for Networks:
    192.168.2.0
    192.168.12.0/30
    192.168.22.0/30
    192.168.23.0/30
  Routing Information Sources:
    Gateway         Distance      Last Update
    (this router)         90      00:08:52
    192.168.12.1          90      00:03:05
    192.168.23.2          90      00:03:05
  Distance: internal 90 external 170
          
R2#

R1#sh ip route eigrp | i 0.0.0.0
Gateway of last resort is 192.168.12.2 to network 0.0.0.0
D*EX  0.0.0.0/0 [170/3139840] via 192.168.12.2, 00:02:38, Serial0/0/1

Подгонка EIGRP

Настройте параметры использования пропускной способности для EIGRP.

R1(config)#int se0/0/1
R1(config-if)#ip bandwidth-percent eigrp 1 75
R1(config-if)#int se0/0/0
R1(config-if)#ip bandwidth-percent eigrp 1 40
R1(config-if)#end

R2(config)#int se0/1/1
R2(config-if)#ip bandwidth-percent eigrp 1 75
R2(config-if)#end

R3(config)#int se0/3/0
R3(config-if)#ip bandwidth-percent eigrp 1 40
R3(config-if)#end

Настройте интервал отправки пакетов приветствия (hello) и таймер удержания для EIGRP.

R2#show ip eigrp interfaces detail
IP-EIGRP interfaces for process 1
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Fa0/0              0        0/0         0       0/1            0           0
  Hello interval is 5 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 0/0
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use multicast
Se0/1/1            1        0/0        13       0/15          83           0
  Hello interval is 5 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 58/55
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 4
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use unicast
Se0/1/0            1        0/0        14       5/190        250           0
  Hello interval is 5 sec
  Next xmit serial <none>
          
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 58/51
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 4
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use unicast
Lo1                0        0/0         0       0/1            0           0
  Hello interval is 5 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 0/0
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use multicast

Значение таймера приветствия по умолчанию 5s
Значение таймера удержания по умолчанию - не представлено.

R1#show ip eigrp interfaces detail
EIGRP-IPv4 Interfaces for AS(1)
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Gi0/0              0        0/0         0       0/1            0           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 0/0
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Topology-ids on interface - 0 
  Authentication mode is not set

Значение таймера приветствия по умолчанию 5s
Значение таймера удержания по умолчанию 15s


1(config)#int se0/0/1
R1(config-if)#ip hello-interval eigrp 1 60
R1(config-if)#
*Jun 18 12:01:24.251: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is down: Interface PEER-TERMINATION received
R1(config-if)#ip hold-time eigrp 1 180
R1(config-if)#
*Jun 18 12:01:28.795: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is up: new adjacency
R1(config-if)#int se0/0/0
R1(config-if)#
*Jun 18 12:01:43.935: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is down: Interface PEER-TERMINATION received
R1(config-if)#ip hello-interval eigrp 1 60
R1(config-if)#ip hello-interval eigrp 1 60
*Jun 18 12:01:48.699: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (Serial0/0/1) is up: new adjacency
R1(config-if)#ip hold-time eigrp 1 180    
R1(config-if)#end

R2(config)#int se0/1/1
R2(config-if)#ip hello-interval eigrp 1 60
R2(config-if)#ip hold-time eigrp 1 180
R2(config-if)#int se0/1/0                 
R2(config-if)#ip hello-interval eigrp 1 60
*Jan  1 01:17:39.195: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.1 (Serial0/1/1) is down: Interface Goodbye received
R2(config-if)#ip hello-interval eigrp 1 60
R2(config-if)#ip hold-time eigrp 1 180    
R

R3(config)#int se0/3/0       
R3(config-if)#ip hello-interval eigrp 1 60
R3(config-if)#ip hold-time eigrp 1 180
R3(config-if)#int se0/3/1
R3(config-if)#ip hello-interval eigrp 1 60
R3(config-if)#ip hold-time eigrp 1 180    


R2#show ip eigrp interfaces detail
IP-EIGRP interfaces for process 1
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Fa0/0              0        0/0         0       0/1            0           0
  Hello interval is 5 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 0/0
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use multicast
Se0/1/1            1        0/0        14       0/15          71           0
  Hello interval is 60 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 71/75
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 5
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use unicast
Se0/1/0            1        0/0        20       5/190        258           0
  Hello interval is 60 sec
  Next xmit serial <none>
          
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 74/67
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 5
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use unicast
Lo1                0        0/0         0       0/1            0           0
  Hello interval is 5 sec
  Next xmit serial <none>
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 0/0
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Authentication mode is not set
  Use multicast

