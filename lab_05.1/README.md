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

Результат изменений:

```shell
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

Файл изменений конфигурации маршрутизатора R2: [R2.conf](configs/R2_conf_part1.txt)

Результат изменений:

```shell
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

Файл изменений конфигурации маршрутизатора R3: [R3.conf](configs/R3_conf_part1.txt)

Результат изменений:

```shell
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

Файл изменений конфигурации маршрутизатора R1: [](configs/R1_conf_part2.txt)


## Часть 3. Изменение назначений идентификаторов маршрутизаторов

## Часть 4. Настройка пассивных интерфейсов OSPF

## Часть 5. Изменение метрик OSPF
