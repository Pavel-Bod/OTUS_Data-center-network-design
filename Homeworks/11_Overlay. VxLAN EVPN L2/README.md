# Домашнее задание №5
## Overlay. VxLAN EVPN L2
### Цель:
- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
- Проверить связанность между устройствами
### Выполнение:
#### Собранная схема сети
![](images/sh.png)

#### Конфигурация оборудования

- [Leaf-1](config/Leaf-1.conf)

```
service routing protocols model multi-agent
!
vlan 10,20
!
interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 10
!
interface Loopback100
   ip address 10.42.204.1/32
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan learn-restrict any
!
router bgp 65501
   router-id 10.42.201.1
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
   neighbor 10.42.200.1 peer group evpn
   neighbor 10.42.200.2 peer group evpn
   neighbor 10.42.202.0 peer group spines
   neighbor 10.42.202.0 bfd
   neighbor 10.42.202.0 password 7 ibb0kmOVqswUe/kiMbQcQg==
   neighbor 10.42.203.0 peer group spines
   neighbor 10.42.203.0 bfd
   neighbor 10.42.203.0 password 7 ibb0kmOVqswUe/kiMbQcQg==
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
      network 10.42.201.1/32
      network 10.42.204.1/32
```

- [Leaf-2](config/Leaf-2.conf)

```
service routing protocols model multi-agent
!
vlan 10,20
!
interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 10
!
interface Loopback100
   ip address 10.42.204.2/32
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan learn-restrict any
!
router bgp 65502
   router-id 10.42.201.2
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
   neighbor 10.42.200.1 peer group evpn
   neighbor 10.42.200.2 peer group evpn
   neighbor 10.42.202.2 peer group spines
   neighbor 10.42.202.2 bfd
   neighbor 10.42.202.2 password 7 2anhXaro2sOjrW/UwD+rrg==
   neighbor 10.42.203.2 peer group spines
   neighbor 10.42.203.2 bfd
   neighbor 10.42.203.2 password 7 2anhXaro2sOjrW/UwD+rrg==
   !
   vlan 10
      rd 65502:100010
      route-target both 10:100010
      redistribute learned
   !
   vlan 20
      rd 65502:100020
      route-target both 20:100020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      network 10.42.201.2/32
      network 10.42.204.2/32
```

- [Leaf-3](config/Leaf-3.conf)

```
service routing protocols model multi-agent
!
vlan 110,120
!
interface Ethernet7
   switchport access vlan 120
!
interface Ethernet8
   switchport access vlan 110
!
interface Loopback100
   ip address 10.42.204.3/32
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 110 vni 100010
   vxlan vlan 120 vni 100020
   vxlan learn-restrict any
!
router bgp 65503
   router-id 10.42.201.3
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
   neighbor 10.42.200.1 peer group evpn
   neighbor 10.42.200.2 peer group evpn
   neighbor 10.42.202.4 peer group spines
   neighbor 10.42.202.4 bfd
   neighbor 10.42.202.4 password 7 UAz8iCGpVsnDNsqXDgEJtw==
   neighbor 10.42.203.4 peer group spines
   neighbor 10.42.203.4 bfd
   neighbor 10.42.203.4 password 7 UAz8iCGpVsnDNsqXDgEJtw==
   !
   vlan 110
      rd 65503:100010
      route-target both 10:100010
      redistribute learned
   !
   vlan 120
      rd 65503:100020
      route-target both 20:100020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      network 10.42.201.3/32
      network 10.42.204.3/32
```

- [Spine-1](config/Spine-1.conf)

```
service routing protocols model multi-agent
!
peer-filter leaf_as
   10 match as-range 65501-65512 result accept
!
router bgp 65500
   router-id 10.42.200.1
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 12
   bgp listen range 10.42.201.0/24 peer-group evpn peer-filter leaf_as
   bgp listen range 10.42.202.0/24 peer-group leafes peer-filter leaf_as
   neighbor evpn peer group
   neighbor evpn next-hop-unchanged
   neighbor evpn update-source Loopback1
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor evpn maximum-routes 12000 warning-only
   neighbor leafes peer group
   neighbor leafes bfd
   neighbor leafes password 7 1hwb917VqCw1BQE1hPI0Hg==
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor leafes activate
      network 10.42.200.1/32
```

- [Spine-2](config/Spine-2.conf)

```
service routing protocols model multi-agent
!
peer-filter leaf_as
   10 match as-range 65501-65512 result accept
!
router bgp 65500
   router-id 10.42.200.2
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 12
   bgp listen range 10.42.201.0/24 peer-group evpn peer-filter leaf_as
   bgp listen range 10.42.203.0/24 peer-group leafes peer-filter leaf_as
   neighbor evpn peer group
   neighbor evpn next-hop-unchanged
   neighbor evpn update-source Loopback1
   neighbor evpn ebgp-multihop 3
   neighbor evpn send-community extended
   neighbor evpn maximum-routes 12000 warning-only
   neighbor leafes peer group
   neighbor leafes bfd
   neighbor leafes password 7 1hwb917VqCw1BQE1hPI0Hg==
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor leafes activate
      network 10.42.200.2/32
```
---
#### Проверка связности 

- Spine-1

```
Spine-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.42.200.1, local AS number 65500
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.42.202.1      4  65501            873       863    0    0 00:42:49 Estab   1      1
  10.42.202.3      4  65502            691       691    0    0 00:34:11 Estab   1      1
  10.42.202.5      4  65503            691       690    0    0 00:34:09 Estab   1      1

Spine-1#sh ip route

 C        10.42.200.1/32 is directly connected, Loopback1
 B E      10.42.201.1/32 [20/0] via 10.42.202.1, Ethernet1
 B E      10.42.201.2/32 [20/0] via 10.42.202.3, Ethernet2
 B E      10.42.201.3/32 [20/0] via 10.42.202.5, Ethernet3
 C        10.42.202.0/31 is directly connected, Ethernet1
 C        10.42.202.2/31 is directly connected, Ethernet2
 C        10.42.202.4/31 is directly connected, Ethernet3
```

- Spine-2

```
Spine-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.42.200.2, local AS number 65500
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.42.203.1      4  65501            759       764    0    0 00:37:40 Estab   1      1
  10.42.203.3      4  65502            740       741    0    0 00:36:43 Estab   1      1
  10.42.203.5      4  65503            729       730    0    0 00:36:10 Estab   1      1

Spine-2#sh ip route

 C        10.42.200.2/32 is directly connected, Loopback1
 B E      10.42.201.1/32 [20/0] via 10.42.203.1, Ethernet1
 B E      10.42.201.2/32 [20/0] via 10.42.203.3, Ethernet2
 B E      10.42.201.3/32 [20/0] via 10.42.203.5, Ethernet3
 C        10.42.203.0/31 is directly connected, Ethernet1
 C        10.42.203.2/31 is directly connected, Ethernet2
 C        10.42.203.4/31 is directly connected, Ethernet3
```

- Leaf-1

```
Leaf-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.42.201.1, local AS number 65501
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.42.202.0      4  65500           3529      3544    0    0 00:45:16 Estab   3      3
  10.42.203.0      4  65500           2346      2350    0    0 00:38:49 Estab   3      3

Leaf-1#sh ip route

 B E      10.42.200.1/32 [20/0] via 10.42.202.0, Ethernet1
 B E      10.42.200.2/32 [20/0] via 10.42.203.0, Ethernet2
 C        10.42.201.1/32 is directly connected, Loopback2
 B E      10.42.201.2/32 [20/0] via 10.42.202.0, Ethernet1
                                via 10.42.203.0, Ethernet2
 B E      10.42.201.3/32 [20/0] via 10.42.202.0, Ethernet1
                                via 10.42.203.0, Ethernet2
 C        10.42.202.0/31 is directly connected, Ethernet1
 C        10.42.203.0/31 is directly connected, Ethernet2

Leaf-1#ping 10.42.201.2 source 10.42.201.1
PING 10.42.201.2 (10.42.201.2) from 10.42.201.1 : 72(100) bytes of data.
80 bytes from 10.42.201.2: icmp_seq=1 ttl=63 time=13.5 ms
80 bytes from 10.42.201.2: icmp_seq=2 ttl=63 time=8.32 ms
80 bytes from 10.42.201.2: icmp_seq=3 ttl=63 time=8.08 ms
80 bytes from 10.42.201.2: icmp_seq=4 ttl=63 time=10.3 ms
80 bytes from 10.42.201.2: icmp_seq=5 ttl=63 time=9.21 ms

--- 10.42.201.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 8.081/9.903/13.563/1.995 ms, ipg/ewma 13.153/11.703 ms

Leaf-1#ping 10.42.201.3 source 10.42.201.1
PING 10.42.201.3 (10.42.201.3) from 10.42.201.1 : 72(100) bytes of data.
80 bytes from 10.42.201.3: icmp_seq=1 ttl=63 time=10.7 ms
80 bytes from 10.42.201.3: icmp_seq=2 ttl=63 time=8.07 ms
80 bytes from 10.42.201.3: icmp_seq=3 ttl=63 time=7.87 ms

--- 10.42.201.3 ping statistics ---
5 packets transmitted, 3 received, 40% packet loss, time 43ms
rtt min/avg/max/mdev = 7.871/8.898/10.748/1.312 ms, ipg/ewma 10.923/10.096 ms
```

- Leaf-2

```
Leaf-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.42.201.2, local AS number 65502
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.42.202.2      4  65500           2024      2027    0    0 00:38:00 Estab   3      3
  10.42.203.2      4  65500           2235      2238    0    0 00:39:13 Estab   3      3

Leaf-2#sh ip route

 B E      10.42.200.1/32 [20/0] via 10.42.202.2, Ethernet1
 B E      10.42.200.2/32 [20/0] via 10.42.203.2, Ethernet2
 B E      10.42.201.1/32 [20/0] via 10.42.202.2, Ethernet1
                                via 10.42.203.2, Ethernet2
 C        10.42.201.2/32 is directly connected, Loopback2
 B E      10.42.201.3/32 [20/0] via 10.42.202.2, Ethernet1
                                via 10.42.203.2, Ethernet2
 C        10.42.202.2/31 is directly connected, Ethernet1
 C        10.42.203.2/31 is directly connected, Ethernet2

Leaf-2#ping 10.42.201.1 source 10.42.201.2
PING 10.42.201.1 (10.42.201.1) from 10.42.201.2 : 72(100) bytes of data.
80 bytes from 10.42.201.1: icmp_seq=1 ttl=63 time=21.2 ms
80 bytes from 10.42.201.1: icmp_seq=2 ttl=63 time=15.6 ms
80 bytes from 10.42.201.1: icmp_seq=3 ttl=63 time=34.4 ms
80 bytes from 10.42.201.1: icmp_seq=4 ttl=63 time=17.7 ms
80 bytes from 10.42.201.1: icmp_seq=5 ttl=63 time=6.94 ms

--- 10.42.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 96ms
rtt min/avg/max/mdev = 6.941/19.223/34.426/8.952 ms, pipe 2, ipg/ewma 24.030/19.893 ms

Leaf-2#ping 10.42.201.3 source 10.42.201.2
PING 10.42.201.3 (10.42.201.3) from 10.42.201.2 : 72(100) bytes of data.
80 bytes from 10.42.201.3: icmp_seq=1 ttl=63 time=13.1 ms
80 bytes from 10.42.201.3: icmp_seq=2 ttl=63 time=12.0 ms
80 bytes from 10.42.201.3: icmp_seq=3 ttl=63 time=7.47 ms
80 bytes from 10.42.201.3: icmp_seq=4 ttl=63 time=8.32 ms
80 bytes from 10.42.201.3: icmp_seq=5 ttl=63 time=8.34 ms

--- 10.42.201.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 7.474/9.872/13.124/2.280 ms, ipg/ewma 13.181/11.374 ms
```

- Leaf-3

```
Leaf-3#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.42.201.3, local AS number 65503
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.42.202.4      4  65500           2005      2007    0    0 00:38:47 Estab   3      3
  10.42.203.4      4  65500           2205      2208    0    0 00:39:29 Estab   3      3

Leaf-3#sh ip route

 B E      10.42.200.1/32 [20/0] via 10.42.202.4, Ethernet1
 B E      10.42.200.2/32 [20/0] via 10.42.203.4, Ethernet2
 B E      10.42.201.1/32 [20/0] via 10.42.202.4, Ethernet1
                                via 10.42.203.4, Ethernet2
 B E      10.42.201.2/32 [20/0] via 10.42.202.4, Ethernet1
                                via 10.42.203.4, Ethernet2
 C        10.42.201.3/32 is directly connected, Loopback2
 C        10.42.202.4/31 is directly connected, Ethernet1
 C        10.42.203.4/31 is directly connected, Ethernet2

Leaf-3#ping 10.42.201.1 source 10.42.201.3
PING 10.42.201.1 (10.42.201.1) from 10.42.201.3 : 72(100) bytes of data.
80 bytes from 10.42.201.1: icmp_seq=1 ttl=63 time=9.88 ms
80 bytes from 10.42.201.1: icmp_seq=2 ttl=63 time=16.7 ms
80 bytes from 10.42.201.1: icmp_seq=3 ttl=63 time=12.0 ms
80 bytes from 10.42.201.1: icmp_seq=4 ttl=63 time=12.1 ms
80 bytes from 10.42.201.1: icmp_seq=5 ttl=63 time=8.95 ms

--- 10.42.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 8.950/11.969/16.768/2.704 ms, pipe 2, ipg/ewma 11.775/10.806 ms

Leaf-3#ping 10.42.201.2 source 10.42.201.3
PING 10.42.201.2 (10.42.201.2) from 10.42.201.3 : 72(100) bytes of data.
80 bytes from 10.42.201.2: icmp_seq=1 ttl=63 time=9.03 ms
80 bytes from 10.42.201.2: icmp_seq=2 ttl=63 time=8.14 ms
80 bytes from 10.42.201.2: icmp_seq=3 ttl=63 time=8.63 ms
80 bytes from 10.42.201.2: icmp_seq=4 ttl=63 time=11.3 ms
80 bytes from 10.42.201.2: icmp_seq=5 ttl=63 time=12.5 ms

--- 10.42.201.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 8.144/9.942/12.580/1.711 ms, pipe 2, ipg/ewma 10.513/9.617 ms

```

#### Проверка bfd

- Spine-1

```
Spine-1#sh bfd peers
VRF name: default
-----------------
DstAddr         MyDisc   YourDisc  Interface/Transport    Type          LastUp
----------- ---------- ----------- -------------------- ------- ---------------
10.42.202.1 3776581285  710359089        Ethernet1(13)  normal  05/29/24 14:14
10.42.202.3  350334203 1621072084        Ethernet2(14)  normal  05/29/24 14:22
10.42.202.5 2562599469  522653439        Ethernet3(15)  normal  05/29/24 14:22

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```

- Spine-2

```
Spine-2#sh bfd peers
VRF name: default
-----------------
DstAddr         MyDisc   YourDisc  Interface/Transport    Type          LastUp
----------- ---------- ----------- -------------------- ------- ---------------
10.42.203.1 2388225729 2651731160        Ethernet1(13)  normal  05/29/24 14:20
10.42.203.3 2775006766  640327943        Ethernet2(14)  normal  05/29/24 14:21
10.42.203.5 1999665187  814292712        Ethernet3(15)  normal  05/29/24 14:22

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
```

- Leaf-1

```
Leaf-1#sh bfd peers
VRF name: default
-----------------
DstAddr         MyDisc   YourDisc  Interface/Transport    Type          LastUp
----------- ---------- ----------- -------------------- ------- ---------------
10.42.202.0  710359089 3776581285        Ethernet1(13)  normal  05/29/24 14:15
10.42.203.0 2651731160 2388225729        Ethernet2(14)  normal  05/29/24 14:21

         LastDown            LastDiag    State
-------------------- ------------------- -----
   05/29/24 14:12       No Diagnostic       Up
   05/29/24 14:20       No Diagnostic       Up
```

- Leaf-2

```
Leaf-2#sh bfd peers
VRF name: default
-----------------
DstAddr         MyDisc   YourDisc  Interface/Transport    Type          LastUp
----------- ---------- ----------- -------------------- ------- ---------------
10.42.202.2 1621072084  350334203        Ethernet1(13)  normal  05/29/24 14:22
10.42.203.2  640327943 2775006766        Ethernet2(14)  normal  05/29/24 14:21

         LastDown            LastDiag    State
-------------------- ------------------- -----
   05/29/24 14:11       No Diagnostic       Up
   05/29/24 14:20       No Diagnostic       Up
```

- Leaf-3

```
Leaf-3#sh bfd peers
VRF name: default
-----------------
DstAddr        MyDisc    YourDisc  Interface/Transport    Type          LastUp
----------- ---------- ----------- -------------------- ------- ---------------
10.42.202.4 522653439  2562599469        Ethernet1(13)  normal  05/29/24 14:22
10.42.203.4 814292712  1999665187        Ethernet2(14)  normal  05/29/24 14:22

         LastDown            LastDiag    State
-------------------- ------------------- -----
   05/29/24 14:11       No Diagnostic       Up
   05/29/24 14:20       No Diagnostic       Up
```
