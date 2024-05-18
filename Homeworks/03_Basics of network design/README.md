# Домашнее задание №1
## Проектирование адресного пространства
### Цель:
- Собрать схему CLOS;
- Распределить адресное пространство;
### Выполнение:
#### Собранная схема сети
![](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/images/sh.png)
#### Адресация
##### Spine1		
Интерфейс | ip-адрес | Назначение
--- | --- | ---
loop1 | 10.42.200.1/32 | 
Eth1 | 10.42.202.0/31 | to Leaf1
Eth2 | 10.42.202.2/31 | to Leaf2
Eth3 | 10.42.202.4/31 | to Leaf3

##### Spine2		
Интерфейс | ip-адрес | Назначение
--- | --- | ---
loop1 | 10.42.200.2/32	
Eth1 | 10.42.203.0/31 | to Leaf1
Eth2 | 10.42.203.2/31 | to Leaf2
Eth3 | 10.42.203.4/31 | to Leaf3
		
##### Leaf1		
Интерфейс | ip-адрес | Назначение
--- | --- | ---
loop2 | 10.42.201.1/32 | 
Eth1 | 10.42.202.1/31 | to Spine1
Eth2 | 10.42.203.1/31 | to Spine2
		
##### Leaf2		
Интерфейс | ip-адрес | Назначение
--- | --- | ---
loop2 | 10.42.201.2/32	
Eth1 | 10.42.202.3/31 | to Spine1
Eth2 | 10.42.203.3/31 | to Spine2
		
##### Leaf3		
Интерфейс | ip-адрес | Назначение
--- | --- | ---
loop2 | 10.42.201.3/32 | 
Eth1 | 10.42.202.5/31 | to Spine1
Eth2 | 10.42.203.5/31 | to Spine2

#### Конфигурация оборудования
- [Leaf-1](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/config/Leaf-1.conf)

```
hostname Leaf-1

interface Ethernet1
   no switchport
   ip address 10.42.202.1/31

interface Ethernet2
   no switchport
   ip address 10.42.203.1/31

interface Loopback2
   ip address 10.42.201.1/32
```
   
- [Leaf-2](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/config/Leaf-2.conf)

```
hostname Leaf-2

interface Ethernet1
   no switchport
   ip address 10.42.202.3/31

interface Ethernet2
   no switchport
   ip address 10.42.203.3/31

interface Loopback2
   ip address 10.42.201.2/32
```
- [Leaf-3](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/config/Leaf-3.conf)

```
hostname Leaf-3

interface Ethernet1
   no switchport
   ip address 10.42.202.5/31

interface Ethernet2
   no switchport
   ip address 10.42.203.5/31

interface Loopback2
   ip address 10.42.201.3/32
```
- [Spine-1](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/config/Spine-1.conf)

```
hostname Spine-1

interface Ethernet1
   no switchport
   ip address 10.42.202.0/31

interface Ethernet2
   no switchport
   ip address 10.42.202.2/31

interface Ethernet3
   no switchport
   ip address 10.42.202.4/31

interface Loopback1
   ip address 10.42.200.1/32
```
- [Spine-2](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/blob/main/Homeworks/03_Basics%20of%20network%20design/config/Spine-2.conf)

```
hostname Spine-2

interface Ethernet1
   no switchport
   ip address 10.42.203.0/31

interface Ethernet2
   no switchport
   ip address 10.42.203.2/31

interface Ethernet3
   no switchport
   ip address 10.42.203.4/31

interface Loopback1
   ip address 10.42.200.2/32
```
#### Проверка доступности

- Spine-1

```
Spine-1#ping 10.42.202.1
PING 10.42.202.1 (10.42.202.1) 72(100) bytes of data.
80 bytes from 10.42.202.1: icmp_seq=1 ttl=64 time=44.3 ms
80 bytes from 10.42.202.1: icmp_seq=2 ttl=64 time=36.3 ms
80 bytes from 10.42.202.1: icmp_seq=3 ttl=64 time=30.5 ms
80 bytes from 10.42.202.1: icmp_seq=4 ttl=64 time=22.8 ms
80 bytes from 10.42.202.1: icmp_seq=5 ttl=64 time=14.8 ms

--- 10.42.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 14.859/29.808/44.376/10.282 ms, pipe 5, ipg/ewma 11.074/36.341 ms
Spine-1#ping 10.42.202.3
PING 10.42.202.3 (10.42.202.3) 72(100) bytes of data.
80 bytes from 10.42.202.3: icmp_seq=1 ttl=64 time=31.0 ms
80 bytes from 10.42.202.3: icmp_seq=2 ttl=64 time=21.6 ms
80 bytes from 10.42.202.3: icmp_seq=3 ttl=64 time=17.0 ms
80 bytes from 10.42.202.3: icmp_seq=4 ttl=64 time=4.31 ms
80 bytes from 10.42.202.3: icmp_seq=5 ttl=64 time=3.44 ms

--- 10.42.202.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 74ms
rtt min/avg/max/mdev = 3.441/15.485/31.004/10.496 ms, pipe 3, ipg/ewma 18.657/22.518 ms
Spine-1#ping 10.42.202.5
PING 10.42.202.5 (10.42.202.5) 72(100) bytes of data.
80 bytes from 10.42.202.5: icmp_seq=1 ttl=64 time=52.0 ms
80 bytes from 10.42.202.5: icmp_seq=2 ttl=64 time=44.8 ms
80 bytes from 10.42.202.5: icmp_seq=3 ttl=64 time=49.2 ms
80 bytes from 10.42.202.5: icmp_seq=4 ttl=64 time=42.0 ms
80 bytes from 10.42.202.5: icmp_seq=5 ttl=64 time=34.7 ms

--- 10.42.202.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 34.736/44.596/52.000/6.008 ms, pipe 5, ipg/ewma 10.612/47.901 ms
```

- Spine-2

```
Spine-2#ping 10.42.203.1
PING 10.42.203.1 (10.42.203.1) 72(100) bytes of data.
80 bytes from 10.42.203.1: icmp_seq=1 ttl=64 time=33.5 ms
80 bytes from 10.42.203.1: icmp_seq=2 ttl=64 time=26.2 ms
80 bytes from 10.42.203.1: icmp_seq=3 ttl=64 time=19.2 ms
80 bytes from 10.42.203.1: icmp_seq=4 ttl=64 time=4.74 ms
80 bytes from 10.42.203.1: icmp_seq=5 ttl=64 time=4.61 ms

--- 10.42.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 86ms
rtt min/avg/max/mdev = 4.612/17.692/33.576/11.548 ms, pipe 3, ipg/ewma 21.645/24.820 ms
Spine-2#ping 10.42.203.3
PING 10.42.203.3 (10.42.203.3) 72(100) bytes of data.
80 bytes from 10.42.203.3: icmp_seq=1 ttl=64 time=10.6 ms
80 bytes from 10.42.203.3: icmp_seq=2 ttl=64 time=4.09 ms
80 bytes from 10.42.203.3: icmp_seq=3 ttl=64 time=3.28 ms
80 bytes from 10.42.203.3: icmp_seq=4 ttl=64 time=3.24 ms
80 bytes from 10.42.203.3: icmp_seq=5 ttl=64 time=3.39 ms

--- 10.42.203.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 3.245/4.935/10.661/2.879 ms, ipg/ewma 9.905/7.685 ms
Spine-2#ping 10.42.203.5
PING 10.42.203.5 (10.42.203.5) 72(100) bytes of data.
80 bytes from 10.42.203.5: icmp_seq=1 ttl=64 time=22.3 ms
80 bytes from 10.42.203.5: icmp_seq=2 ttl=64 time=9.90 ms
80 bytes from 10.42.203.5: icmp_seq=3 ttl=64 time=4.06 ms
80 bytes from 10.42.203.5: icmp_seq=4 ttl=64 time=4.09 ms
80 bytes from 10.42.203.5: icmp_seq=5 ttl=64 time=5.13 ms

--- 10.42.203.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 72ms
rtt min/avg/max/mdev = 4.062/9.116/22.392/6.979 ms, pipe 2, ipg/ewma 18.072/15.433 ms

```
