## Настройка расширенных сетей VLAN, VTP и DTP

### 1. Часть 1. Настройка VTP

#### Коммутатор S1
```
S1(config)#do show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0008.302d.4580
Configuration last modified by 0.0.0.0 at 3-1-93 00:40:06
Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 255
Number of existing VLANs          : 9
Configuration Revision            : 8
MD5 digest                        : 0x16 0x68 0xF2 0x0B 0x67 0x31 0x81 0x1B 
                                    0x34 0xDB 0xE5 0xB5 0x68 0x63 0x5A 0x70 
```
#### Коммутатор S2
```
```
#### Коммутатор S3
```
S3#show vtp status
VTP Version                     : 2
Configuration Revision          : 8
Maximum VLANs supported locally : 250
Number of existing VLANs        : 9
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x16 0x68 0xF2 0x0B 0x67 0x31 0x81 0x1B 
Configuration last modified by 0.0.0.0 at 3-1-93 00:40:06
S3# 
```

### Часть 2. Настройка DTP

#### S1
```
S1#sh int f0/1 switchport 
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk
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
Administrative private-vlan trunk encapsulation: dot1q
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
S1#

```

### Часть 3. Добавление сетей VLAN и назначение портов
### Часть 4. Настройка расширенной сети VLAN

