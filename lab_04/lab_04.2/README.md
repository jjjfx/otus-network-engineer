# Настройка EtherChannel

### Топология:
![](images/otus-topology.png)

### Таблица адресации:

| Устройство | Интерфейс     | IP-адрес      | Маска подсети   |
|:-----------|:--------------|:--------------|:----------------|
| S1         | VLAN99        | 192.168.99.11 | 255.255.255.0   |
| S2         | VLAN99        | 192.168.99.12 | 255.255.255.0   |
| S3         | VLAN99        | 192.168.99.13 | 255.255.255.0   |
| PC-A       | NIC           | 192.168.10.1  | 255.255.255.0   |
| PC-B       | NIC           | 192.168.10.2  | 255.255.255.0   |
| PC-C       | NIC           | 192.168.10.3  | 255.255.255.0   |

## Часть 1. Создание сети и настройка основных параметров устройства

Файл базовой настройки коммутатора S1: [S1.conf](configs/S1_conf.txt)  
Файл базовой настройки коммутатора S2: [S2.conf](configs/S2_conf.txt)
Файл базовой настройки коммутатора S3: [S3.conf](configs/S3_conf.txt)

## Часть 2. Настройка протокола PAgP

S1:  
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#int range fa0/1 - 2
S1(config-if-range)#channel-group 1 mode desirable
Creating a port-channel interface Port-channel 1
S1(config-if-range)#
*Mar  1 00:20:43.543: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
*Mar  1 00:20:43.569: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
S1(config-if-range)#no shutdow
*Mar  1 00:20:52.276: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
*Mar  1 00:20:53.216: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
S1(config-if-range)#no shutdown
S1(config-if-range)#
*Mar  1 00:21:37.348: %LINK-3-UPDOWN: Interface Port-channel1, changed state to up
*Mar  1 00:21:38.355: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
S1(config-if-range)#
```

S3:  
```
S3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#int range fa0/3 - 4
S3(config-if-range)#channel-group 1 mode desirable
Creating a port-channel interface Port-channel 1
S3(config-if-range)#
00:22:38: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
00:22:38: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
S3(config-if-range)#no shutdown
00:22:41: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
00:22:41: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
S3(config-if-range)#no shutdown
S3(config-if-range)#
00:22:42: %LINK-3-UPDOWN: Interface Port-channel1, changed state to up
00:22:43: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
S3(config-if-range)#
```

```
S1#sh run int fa0/1
Building configuration...
Current configuration : 93 bytes
!
interface FastEthernet0/1
 description -- to S3 fa0/3
 channel-group 1 mode desirable
end
S1#sh int fa0/1 switchport
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk (member of bundle Po1)
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vl encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
S1#sh etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        M - not in use, minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port
Number of channel-groups in use: 1
Number of aggregators:           1
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Fa0/1(P)    Fa0/2(P)    
```

```
S3#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        u - unsuitable for bundling
        U - in use      f - failed to allocate aggregator
        d - default port
Number of channel-groups in use: 1
Number of aggregators:           1
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Fa0/3(Pd)   Fa0/4(P)    
```

Символы SU в итоговом выводе означает, что объединение интерфейсов выполнено на 2-м уровне OSI,  
символ P - признак принадлежности интерфейса в аггрегированному каналу.

Переводим Po1 в транковый режим:  
```
S1(config)#int po1
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 99
S1(config-if)#end

S3(config)#int po1
S3(config-if)#switchport mode trunk
S3(config-if)#switchport trunk native vlan 99
00:34:12: %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (1), with S1 FastEthernet0/1 (99).
00:34:12: %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (1), with S1 FastEthernet0/2 (99).
S3(config-if)#switchport trunk native vlan 99
S3(config-if)#end
S3#
00:34:18: %SYS-5-CONFIG_I: Configured from console by console
S3#
00:34:30: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Port-channel1 on VLAN0099. Port consistency restored.
00:34:30: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Port-channel1 on VLAN0001. Port consistency restored.
```



```
S1#sh run int fa0/1
Building configuration...
Current configuration : 149 bytes
!
interface FastEthernet0/1
 description -- to S3 fa0/3
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
end
S1#sh run int fa0/2
Building configuration...
Current configuration : 149 bytes
!
interface FastEthernet0/2
 description -- to S3 fa0/4
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
end

S3#sh run int fa0/3
Building configuration...
Current configuration : 149 bytes
!
interface FastEthernet0/3
 description -- to S1 fa0/1
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
end
S3#sh run int fa0/4
Building configuration...
Current configuration : 149 bytes
!
interface FastEthernet0/4
 description -- to S1 fa0/2
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
end
```

Фиксируем синхронизацию параметров интерфейсов, включенных в Po1, с параметрами самого Po1.

```
S1#sh int trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/3       auto             802.1q         trunking      1
Fa0/4       auto             802.1q         trunking      1
Po1         on               802.1q         trunking      99
Port        Vlans allowed on trunk
Fa0/3       1-4094
Fa0/4       1-4094
Po1         1-4094
Port        Vlans allowed and active in management domain
Fa0/3       1,10,99
Fa0/4       1,10,99
Po1         1,10,99
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/3       1,10,99
Fa0/4       1,10,99
Po1         1,10,99
S1#sh spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0008.302d.4580
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0008.302d.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/3               Desg FWD 19        128.3    P2p 
Fa0/4               Desg FWD 19        128.4    P2p 
Po1                 Desg FWD 12        128.64   P2p 
          
VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     0008.302d.4580
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0008.302d.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/3               Desg FWD 19        128.3    P2p 
Fa0/4               Desg FWD 19        128.4    P2p 
Fa0/15              Desg FWD 19        128.15   P2p 
Po1                 Desg FWD 12        128.64   P2p 
          
VLAN0099
  Spanning tree enabled protocol ieee
  Root ID    Priority    32867
             Address     0008.302d.4580
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32867  (priority 32768 sys-id-ext 99)
             Address     0008.302d.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/3               Desg FWD 19        128.3    P2p 
Fa0/4               Desg FWD 19        128.4    P2p 
Po1                 Desg FWD 12        128.64   P2p 
S1# 
```

```
S3#sh int trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       desirable    802.1q         trunking      1
Fa0/2       desirable    802.1q         trunking      1
Po1         on           802.1q         trunking      99
Port      Vlans allowed on trunk
Fa0/1       1-4094
Fa0/2       1-4094
Po1         1-4094
Port        Vlans allowed and active in management domain
Fa0/1       1,10,99
Fa0/2       1,10,99
Po1         1,10,99
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,99
Fa0/2       1,10,99
Po1         1,10,99
S3#sh spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0008.302d.4580
             Cost        12
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0011.2142.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p 
Fa0/2            Desg FWD 19        128.2    P2p 
Po1              Root FWD 12        128.65   P2p 
          
VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     0008.302d.4580
             Cost        12
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0011.2142.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p 
Fa0/2            Desg FWD 19        128.2    P2p 
Fa0/15           Desg FWD 19        128.15   P2p 
Po1              Root FWD 12        128.65   P2p 
          
VLAN0099
  Spanning tree enabled protocol ieee
  Root ID    Priority    32867
             Address     0008.302d.4580
             Cost        12
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
  Bridge ID  Priority    32867  (priority 32768 sys-id-ext 99)
             Address     0011.2142.4580
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p 
Fa0/2            Desg FWD 19        128.2    P2p 
Po1              Root FWD 12        128.65   P2p 
S3#
```

## Часть 3. Настройка протокола LACP

```
S1(config)#int range fa0/3 - 4
S1(config-if-range)#switchport mode trunk 
S1(config-if-range)#switchport trunk native vlan 99
S1(config-if-range)#channel-group 2 mode active
Creating a port-channel interface Port-channel 2

S2(config)#int range fa0/3 - 4
S2(config-if-range)#switchport mode trunk
S2(config-if-range)#switchport trunk native vlan 99
S2(config-if-range)#channel-group 2 mode passive
00:49:27: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking FastEthernet0/3 on VLAN0099. Port consistency restored.
00:49:27: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking FastEthernet0/3 on VLAN0001. Port consistency restored.
00:49:27: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking FastEthernet0/4 on VLAN0099. Port consistency restored.
00:49:27: %SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking FastEthernet0/4 on VLAN0001. Port consistency restored.
S2(config-if-range)#channel-group 2 mode passive
Creating a port-channel interface Port-channel 2
```

Проверяем объединение каналов:  
```
S1#sh etherchannel 2 summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)         LACP      Fa0/3(P)    Fa0/4(P)    

S2#sh etherchannel 2 summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)         LACP      Fa0/3(Pd)   Fa0/4(P)    
```

В итоговом выводе зафиксировано состояние канала SU (используется), тип протокола LACP и порты, включенные в данный канал - Fa0/3 и Fa0/4.

