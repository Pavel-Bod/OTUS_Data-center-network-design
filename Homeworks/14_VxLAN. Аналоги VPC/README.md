# Домашнее задание №7
## Overlay. VxLAN EVPN L3
### Цель:
- Подключить клиента 2-я линками к различным Leaf с использование агрегированного канала с LACP
- Настроитm multihoming для работы в Overlay сети с использованием MLAG с поддержкой VXLAN
- Протестировать отказоустойчивость - убедиться, что связнность не теряется при отключении одного из линков
### Выполнение:
#### Собранная схема сети
![](images/sh.png)

#### Конфигурация оборудования

- [Leaf-11](config/Leaf-11.conf)

```
vlan 4090
   name mlag-peer
   trunk group mlag-peer
!
vlan 4091
   name mlag-ibgp
   trunk group mlag-peer
!
vrf instance EVPN
!
vrf instance mgmt
!
interface Port-Channel1
   switchport mode trunk
   mlag 1
!
interface Port-Channel999
   description MLAG Peer
   switchport mode trunk
   switchport trunk group mlag-peer
   spanning-tree link-type point-to-point
!
interface Ethernet1
   no switchport
   ip address 10.42.202.1/31
!
interface Ethernet2
   no switchport
   ip address 10.42.203.1/31
!
interface Ethernet3
   description mlag peer link
   channel-group 999 mode active
!
interface Ethernet4
   description mlag peer link
   channel-group 999 mode active
!
interface Ethernet5
   switchport access vlan 10
   channel-group 1 mode active
!
interface Loopback2
   ip address 10.42.201.11/32
!
interface Loopback100
   ip address 10.42.204.1/32
!
interface Management1
   vrf mgmt
   ip address 10.42.206.1/24
!
interface Vlan10
   vrf EVPN
   ip address 192.168.10.2/24
   ip virtual-router address 192.168.10.1
!
interface Vlan20
   vrf EVPN
   ip address 192.168.20.2/24
   ip virtual-router address 192.168.20.1
!
interface Vlan4090
   no autostate
   ip address 10.42.205.0/31
!
interface Vlan4091
   mtu 9214
   ip address 10.42.207.0/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vrf EVPN vni 9999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:aa:aa:aa:aa:aa
!
ip routing
ip routing vrf EVPN
no ip routing vrf mgmt
!
mlag configuration
   domain-id leafs
   local-interface Vlan4090
   peer-address 10.42.205.1
   peer-address heartbeat 10.42.206.2 vrf mgmt
   peer-link Port-Channel999
   dual-primary detection delay 10 action errdisable all-interfaces
!
router bgp 65501
   router-id 10.42.201.11
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 12
   neighbor evpn peer group
   neighbor evpn remote-as 65500
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor evpn maximum-routes 12000 warning-only
   neighbor spines peer group
   neighbor spines remote-as 65500
   neighbor spines maximum-routes 12000 warning-only
   neighbor underlay_ibgp peer group
   neighbor underlay_ibgp remote-as 65501
   neighbor underlay_ibgp next-hop-self
   neighbor underlay_ibgp maximum-routes 12000 warning-only
   neighbor 10.42.200.1 peer group evpn
   neighbor 10.42.200.2 peer group evpn
   neighbor 10.42.202.0 peer group spines
   neighbor 10.42.202.0 bfd
   neighbor 10.42.202.0 password 7 ibb0kmOVqswUe/kiMbQcQg==
   neighbor 10.42.203.0 peer group spines
   neighbor 10.42.203.0 bfd
   neighbor 10.42.203.0 password 7 ibb0kmOVqswUe/kiMbQcQg==
   neighbor 10.42.207.1 peer group underlay_ibgp
   !
   vlan 10
      rd 65501:100010
      route-target both 10:100010
      redistribute learned
   !
   vlan 20
      rd 65501:100020
      route-target both 20:100020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      neighbor underlay_ibgp activate
      network 10.42.201.1/32
      network 10.42.201.11/32
      network 10.42.204.1/32
   !
   vrf EVPN
      rd 65501:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Leaf-12](config/Leaf-12.conf)

```
vlan 4090
   name mlag-peer
   trunk group mlag-peer
!
vlan 4091
   name mlag-ibgp
   trunk group mlag-peer
!
vrf instance EVPN
!
vrf instance mgmt
!
interface Port-Channel1
   switchport mode trunk
   mlag 1
!
interface Port-Channel999
   description MLAG Peer
   switchport mode trunk
   switchport trunk group mlag-peer
   spanning-tree link-type point-to-point
!
interface Ethernet1
   no switchport
   ip address 10.42.202.7/31
!
interface Ethernet2
   no switchport
   ip address 10.42.203.7/31
!
interface Ethernet3
   description mlag peer link
   channel-group 999 mode active
!
interface Ethernet4
   description mlag peer link
   channel-group 999 mode active
!
interface Ethernet5
   switchport access vlan 10
   channel-group 1 mode active
!
interface Loopback2
   ip address 10.42.201.12/32
!
interface Loopback100
   ip address 10.42.204.1/32
!
interface Management1
   vrf mgmt
   ip address 10.42.206.2/24
!
interface Vlan10
   vrf EVPN
   ip address 192.168.10.3/24
   ip virtual-router address 192.168.10.1
!
interface Vlan20
   vrf EVPN
   ip address 192.168.20.3/24
   ip virtual-router address 192.168.20.1
!
interface Vlan4090
   no autostate
   ip address 10.42.205.1/31
!
interface Vlan4091
   mtu 9214
   ip address 10.42.207.1/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vrf EVPN vni 9999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:aa:aa:aa:aa:aa
!
ip routing
ip routing vrf EVPN
no ip routing vrf mgmt
!
mlag configuration
   domain-id leafs
   local-interface Vlan4090
   peer-address 10.42.205.0
   peer-address heartbeat 10.42.206.1 vrf mgmt
   peer-link Port-Channel999
   dual-primary detection delay 10 action errdisable all-interfaces
!
router bgp 65501
   router-id 10.42.201.12
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 12
   neighbor evpn peer group
   neighbor evpn remote-as 65500
   neighbor evpn update-source Loopback2
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor evpn maximum-routes 12000 warning-only
   neighbor spines peer group
   neighbor spines remote-as 65500
   neighbor spines maximum-routes 12000 warning-only
   neighbor underlay_ibgp peer group
   neighbor underlay_ibgp remote-as 65501
   neighbor underlay_ibgp next-hop-self
   neighbor underlay_ibgp maximum-routes 12000 warning-only
   neighbor 10.42.200.1 peer group evpn
   neighbor 10.42.200.2 peer group evpn
   neighbor 10.42.202.6 peer group spines
   neighbor 10.42.202.6 bfd
   neighbor 10.42.202.6 password 7 qQomqGQnIabJvDCIzGdqnQ==
   neighbor 10.42.203.6 peer group spines
   neighbor 10.42.203.6 bfd
   neighbor 10.42.203.6 password 7 qQomqGQnIabJvDCIzGdqnQ==
   neighbor 10.42.207.0 peer group underlay_ibgp
   !
   vlan 10
      rd 65501:100010
      route-target both 10:100010
      redistribute learned
   !
   vlan 20
      rd 65501:100020
      route-target both 20:100020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      neighbor underlay_ibgp activate
      network 10.42.201.12/32
      network 10.42.204.1/32
   !
   vrf EVPN
      rd 65501:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Server1](config/Server1.conf)

```
vlan 10,20
!
interface Port-Channel1
   switchport mode trunk
!
interface Ethernet1
   channel-group 1 mode active
!
interface Ethernet2
   channel-group 1 mode active
!
interface Vlan10
   ip address 192.168.10.41/24
!
interface Vlan20
   ip address 192.168.20.41/24
!
ip routing
```


---
#### Проверка связности 

- Leaf-11

```

```

- Leaf-12

```

```

- Leaf-3

```

```

- VPC5
```

```

- VPC6
```

```
