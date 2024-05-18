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
Интерфейс	ip-адрес	Назначение
loop1	10.42.200.1/32	
Eth1	10.42.202.0/31	to Leaf1
Eth2	10.42.202.2/31	to Leaf2
Eth3	10.42.202.4/31	to Leaf3

##### Spine2		
Интерфейс	ip-адрес	Назначение
loop1	10.42.200.2/32	
Eth1	10.42.203.0/31	to Leaf1
Eth2	10.42.203.2/31	to Leaf2
Eth3	10.42.203.4/31	to Leaf3
		
##### Leaf1		
Интерфейс	ip-адрес	Назначение
loop2	10.42.201.1/32	
Eth1	10.42.202.1/31	to Spine1
Eth2	10.42.203.1/31	to Spine2
		
##### Leaf2		
Интерфейс	ip-адрес	Назначение
loop2	10.42.201.2/32	
Eth1	10.42.202.3/31	to Spine1
Eth2	10.42.203.3/31	to Spine2
		
##### Leaf3		
Интерфейс	ip-адрес	Назначение
loop2	10.42.201.3/32	
Eth1	10.42.202.5/31	to Spine1
Eth2	10.42.203.5/31	to Spine2

Spine1		
Интерфейс	ip-адрес	Назначение
loop1	10.42.200.1/32	
Eth1	10.42.202.0/31	to Leaf1
Eth2	10.42.202.2/31	to Leaf2
Eth3	10.42.202.4/31	to Leaf3
![image](https://github.com/Pavel-Bod/OTUS_Data-center-network-design/assets/79157757/29776815-cf75-47e8-ae1e-bc05e1d6e9cd)

