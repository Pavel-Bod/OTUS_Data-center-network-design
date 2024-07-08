# Домашнее задание №8
## VXLAN. Routing
### Цель:
- Реализовать передачу суммарных префиксов через EVPN route-type 5
### Выполнение:
#### Собранная схема сети
![](images/sh.png)

#### Конфигурация оборудования

- [Leaf-1](config/Leaf-1.conf)

```
vlan 10,20
!
vrf instance EVPN
!
interface Ethernet1
   no switchport
   ip address 10.42.202.1/31
!
interface Ethernet2
   no switchport
   ip address 10.42.203.1/31
!
interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 10
!
interface Loopback2
   ip address 10.42.201.1/32
!
interface Loopback100
   ip address 10.42.204.1/32
!
interface Vlan10
   vrf EVPN
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf EVPN
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf EVPN vni 9999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:aa:aa:aa:aa:aa
!
ip routing
ip routing vrf EVPN
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
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      network 10.42.201.1/32
      network 10.42.204.1/32
   !
   vrf EVPN
      rd 65501:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Leaf-2](config/Leaf-2.conf)

```
vlan 30,40
!
vrf instance EVPN
!
interface Ethernet1
   no switchport
   ip address 10.42.202.3/31
!
interface Ethernet2
   no switchport
   ip address 10.42.203.3/31
!
interface Ethernet7
   switchport access vlan 40
!
interface Ethernet8
   switchport access vlan 30
!
interface Loopback2
   ip address 10.42.201.2/32
!
interface Loopback100
   ip address 10.42.204.2/32
!
interface Vlan30
   vrf EVPN
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf EVPN
   ip address virtual 192.168.40.1/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf EVPN vni 9999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:bb:bb:bb:bb:bb
!
ip routing
ip routing vrf EVPN
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
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      network 10.42.201.2/32
      network 10.42.204.2/32
   !
   vrf EVPN
      rd 65502:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Leaf-3](config/Leaf-3.conf)

```
vlan 10,50,900
!
vrf instance EVPN
!
interface Ethernet1
   no switchport
   ip address 10.42.202.5/31
!
interface Ethernet2
   no switchport
   ip address 10.42.203.5/31
!
interface Ethernet7
   switchport access vlan 900
   vrf EVPN
!
interface Ethernet8
   switchport access vlan 50
!
interface Loopback2
   ip address 10.42.201.3/32
!
interface Loopback100
   ip address 10.42.204.3/32
!
interface Vlan50
   vrf EVPN
   ip address virtual 192.168.50.1/24
!
interface Vlan900
   vrf EVPN
   ip address 172.16.100.2/30
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf EVPN vni 9999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:cc:cc:cc:cc:cc
!
ip routing
ip routing vrf EVPN
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
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      no neighbor evpn activate
      neighbor spines activate
      network 10.42.201.3/32
      network 10.42.204.3/32
   !
   vrf EVPN
      rd 65503:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      neighbor 172.16.100.1 remote-as 65999
      redistribute connected
      !
      address-family ipv4
         neighbor 172.16.100.1 activate
```

- [Router](config/Router.conf)

```
interface Ethernet1
   no switchport
   ip address 172.16.100.1/30
!
interface Loopback0
   ip address 10.42.201.255/32
!
interface Loopback1
   ip address 8.8.8.8/32
!
interface Loopback2
   ip address 7.7.7.7/32
!
ip routing
!
ip route 0.0.0.0/0 Null0
!
router bgp 65999
   router-id 10.42.201.255
   timers bgp 3 9
   neighbor 172.16.100.2 remote-as 65503
   redistribute connected
   redistribute static
   !
   address-family ipv4
      neighbor 172.16.100.2 activate
      redistribute static
```


---
#### Проверка связности 

- Leaf-1

```
Leaf-1#sh ip route vrf EVPN
VRF: EVPN
 B E      0.0.0.0/0 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

 B E      7.7.7.7/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      8.8.8.8/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      10.42.201.255/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      172.16.100.0/30 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 C        192.168.20.0/24 is directly connected, Vlan20
 B E      192.168.30.0/24 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B E      192.168.40.0/24 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B E      192.168.50.0/24 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

Leaf-1#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.42.201.1, local AS number 65501
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65503:9999 ip-prefix 0.0.0.0/0
                                 10.42.204.3           -       100     0       65500 65503 65999 ?
 *  ec    RD: 65503:9999 ip-prefix 0.0.0.0/0
                                 10.42.204.3           -       100     0       65500 65503 65999 ?
 * >Ec    RD: 65503:9999 ip-prefix 7.7.7.7/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 7.7.7.7/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 8.8.8.8/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 8.8.8.8/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 10.42.201.255/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 10.42.201.255/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 10.42.204.3           -       100     0       65500 65503 i
 * >      RD: 65501:9999 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 65501:9999 ip-prefix 192.168.20.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65502:9999 ip-prefix 192.168.30.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:9999 ip-prefix 192.168.30.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:9999 ip-prefix 192.168.40.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:9999 ip-prefix 192.168.40.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65503:9999 ip-prefix 192.168.50.0/24
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:9999 ip-prefix 192.168.50.0/24
                                 10.42.204.3           -       100     0       65500 65503 i


```

- Leaf-2

```
Leaf-2#sh ip route vrf EVPN
VRF: EVPN
 B E      0.0.0.0/0 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

 B E      7.7.7.7/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      8.8.8.8/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      10.42.201.255/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      172.16.100.0/30 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.10.0/24 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.20.0/24 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        192.168.30.0/24 is directly connected, Vlan30
 C        192.168.40.0/24 is directly connected, Vlan40
 B E      192.168.50.0/24 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

Leaf-2#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.42.201.2, local AS number 65502
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65503:9999 ip-prefix 0.0.0.0/0
                                 10.42.204.3           -       100     0       65500 65503 65999 ?
 *  ec    RD: 65503:9999 ip-prefix 0.0.0.0/0
                                 10.42.204.3           -       100     0       65500 65503 65999 ?
 * >Ec    RD: 65503:9999 ip-prefix 7.7.7.7/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 7.7.7.7/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 8.8.8.8/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 8.8.8.8/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 10.42.201.255/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 *  ec    RD: 65503:9999 ip-prefix 10.42.201.255/32
                                 10.42.204.3           -       100     0       65500 65503 65999 i
 * >Ec    RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65501:9999 ip-prefix 192.168.10.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:9999 ip-prefix 192.168.10.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:9999 ip-prefix 192.168.20.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:9999 ip-prefix 192.168.20.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 * >      RD: 65502:9999 ip-prefix 192.168.30.0/24
                                 -                     -       -       0       i
 * >      RD: 65502:9999 ip-prefix 192.168.40.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65503:9999 ip-prefix 192.168.50.0/24
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:9999 ip-prefix 192.168.50.0/24
                                 10.42.204.3           -       100     0       65500 65503 i

```

- Leaf-3

```
Leaf-3#sh ip route vrf EVPN
VRF: EVPN
Gateway of last resort:
 B E      0.0.0.0/0 [20/0] via 172.16.100.1, Vlan900

 B E      7.7.7.7/32 [20/0] via 172.16.100.1, Vlan900
 B E      8.8.8.8/32 [20/0] via 172.16.100.1, Vlan900
 B E      10.42.201.255/32 [20/0] via 172.16.100.1, Vlan900
 C        172.16.100.0/30 is directly connected, Vlan900
 B E      192.168.10.0/24 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.20.0/24 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.30.0/24 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B E      192.168.40.0/24 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 C        192.168.50.0/24 is directly connected, Vlan50

Leaf-3#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.42.201.3, local AS number 65503
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65503:9999 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65999 ?
 * >      RD: 65503:9999 ip-prefix 7.7.7.7/32
                                 -                     -       100     0       65999 i
 * >      RD: 65503:9999 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65999 i
 * >      RD: 65503:9999 ip-prefix 10.42.201.255/32
                                 -                     -       100     0       65999 i
 * >      RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 -                     -       -       0       i
 *        RD: 65503:9999 ip-prefix 172.16.100.0/30
                                 -                     -       100     0       65999 i
 * >Ec    RD: 65501:9999 ip-prefix 192.168.10.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:9999 ip-prefix 192.168.10.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:9999 ip-prefix 192.168.20.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:9999 ip-prefix 192.168.20.0/24
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65502:9999 ip-prefix 192.168.30.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:9999 ip-prefix 192.168.30.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:9999 ip-prefix 192.168.40.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:9999 ip-prefix 192.168.40.0/24
                                 10.42.204.2           -       100     0       65500 65502 i
 * >      RD: 65503:9999 ip-prefix 192.168.50.0/24
                                 -                     -       -       0       i

```

- VPC1
```
VPC1> ping 192.168.20.11 -c 2

84 bytes from 192.168.20.11 icmp_seq=1 ttl=63 time=280.356 ms
84 bytes from 192.168.20.11 icmp_seq=2 ttl=63 time=6.063 ms

VPC1> ping 192.168.30.11 -c 2

84 bytes from 192.168.30.11 icmp_seq=1 ttl=62 time=237.818 ms
84 bytes from 192.168.30.11 icmp_seq=2 ttl=62 time=16.155 ms

VPC1> ping 192.168.40.11 -c 2

84 bytes from 192.168.40.11 icmp_seq=1 ttl=62 time=94.062 ms
84 bytes from 192.168.40.11 icmp_seq=2 ttl=62 time=14.545 ms

VPC1> ping 192.168.50.11 -c 2

84 bytes from 192.168.50.11 icmp_seq=1 ttl=62 time=317.639 ms
84 bytes from 192.168.50.11 icmp_seq=2 ttl=62 time=18.003 ms

VPC1> ping 8.8.8.8 -c 2

84 bytes from 8.8.8.8 icmp_seq=1 ttl=62 time=35.759 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=23.368 ms

VPC1> ping 7.7.7.7 -c 2

84 bytes from 7.7.7.7 icmp_seq=1 ttl=62 time=19.302 ms
84 bytes from 7.7.7.7 icmp_seq=2 ttl=62 time=19.269 ms
```
