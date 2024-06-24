# Домашнее задание №6
## Overlay. VxLAN EVPN L3
### Цель:
- Настроить маршрутизацию в рамках Overlay между клиентами
- Проверить связанность между устройствами
### Выполнение:
#### Собранная схема сети
![](images/sh.png)

#### Конфигурация оборудования

- [Leaf-1](config/Leaf-1.conf)

```
vrf instance EVPN
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
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
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
   !
   vrf EVPN
      rd 65501:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Leaf-2](config/Leaf-2.conf)

```
vrf instance EVPN
!
interface Loopback100
   ip address 10.42.204.2/32
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
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
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
   !
   vrf EVPN
      rd 65502:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```

- [Leaf-3](config/Leaf-3.conf)

```
vrf instance EVPN
!
interface Loopback100
   ip address 10.42.204.3/32
!
interface Vlan110
   vrf EVPN
   ip address virtual 192.168.10.1/24
!
interface Vlan120
   vrf EVPN
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 110 vni 100010
   vxlan vlan 120 vni 100020
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
   !
   vrf EVPN
      rd 65503:9999
      route-target import evpn 9999:9999
      route-target export evpn 9999:9999
      redistribute connected
```


---
#### Проверка связности 

- Leaf-1

```
Leaf-1#sh ip route vrf EVPN
 B E      192.168.10.21/32 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B E      192.168.10.31/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B E      192.168.20.21/32 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B E      192.168.20.31/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.20.0/24 is directly connected, Vlan20

Leaf-1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.42.201.1, local AS number 65501
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65501:100010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 65501:100010 mac-ip 0050.7966.6806 192.168.10.11
                                 -                     -       -       0       i
 * >Ec    RD: 65502:100010 mac-ip 0050.7966.6807
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100010 mac-ip 0050.7966.6807
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:100010 mac-ip 0050.7966.6807 192.168.10.21
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100010 mac-ip 0050.7966.6807 192.168.10.21
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65503:100010 mac-ip 0050.7966.6808
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100010 mac-ip 0050.7966.6808
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100010 mac-ip 0050.7966.6808 192.168.10.31
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100010 mac-ip 0050.7966.6808 192.168.10.31
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100020 mac-ip 0050.7966.6809
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100020 mac-ip 0050.7966.6809
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100020 mac-ip 0050.7966.6809 192.168.20.31
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100020 mac-ip 0050.7966.6809 192.168.20.31
                                 10.42.204.3           -       100     0       65500 65503 i
 * >      RD: 65501:100020 mac-ip 0050.7966.680a
                                 -                     -       -       0       i
 * >      RD: 65501:100020 mac-ip 0050.7966.680a 192.168.20.11
                                 -                     -       -       0       i
 * >Ec    RD: 65502:100020 mac-ip 0050.7966.680b
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100020 mac-ip 0050.7966.680b
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:100020 mac-ip 0050.7966.680b 192.168.20.21
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100020 mac-ip 0050.7966.680b 192.168.20.21
                                 10.42.204.2           -       100     0       65500 65502 i

Leaf-1#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.42.204.1
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 100010]      [20, 100020]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 9999]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [EVPN, 9999]
  Headend replication flood vtep list is:
    10 10.42.204.2     10.42.204.3
    20 10.42.204.2     10.42.204.3
  Shared Router MAC is 0000.0000.0000

Leaf-1#sh vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI          VLAN       Source       Interface       802.1Q Tag
------------ ---------- ------------ --------------- ----------
100010       10         static       Ethernet8       untagged
                                     Vxlan1          10
100020       20         static       Ethernet7       untagged
                                     Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI        VLAN       VRF        Source
---------- ---------- ---------- ------------
9999       4094       EVPN       evpn


Leaf-1#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6807  EVPN      Vx1  10.42.204.2      1       0:01:37 ago
  10  0050.7966.6808  EVPN      Vx1  10.42.204.3      1       0:01:29 ago
  20  0050.7966.6809  EVPN      Vx1  10.42.204.3      1       0:01:44 ago
  20  0050.7966.680b  EVPN      Vx1  10.42.204.2      1       0:01:12 ago
4094  5000.00cb.38c2  EVPN      Vx1  10.42.204.2      1       1:17:37 ago
4094  5000.00d5.5dc0  EVPN      Vx1  10.42.204.3      1       0:59:29 ago
Total Remote Mac Addresses for this criterion: 6
```

- Leaf-2

```
Leaf-2#sh ip route vrf EVPN
 B E      192.168.10.11/32 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.10.31/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B E      192.168.20.11/32 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.20.31/32 [20/0] via VTEP 10.42.204.3 VNI 9999 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.20.0/24 is directly connected, Vlan20

Leaf-2#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.42.201.2, local AS number 65502
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65501:100010 mac-ip 0050.7966.6806
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100010 mac-ip 0050.7966.6806
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:100010 mac-ip 0050.7966.6806 192.168.10.11
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100010 mac-ip 0050.7966.6806 192.168.10.11
                                 10.42.204.1           -       100     0       65500 65501 i
 * >      RD: 65502:100010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 65502:100010 mac-ip 0050.7966.6807 192.168.10.21
                                 -                     -       -       0       i
 * >Ec    RD: 65503:100010 mac-ip 0050.7966.6808
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100010 mac-ip 0050.7966.6808
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100010 mac-ip 0050.7966.6808 192.168.10.31
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100010 mac-ip 0050.7966.6808 192.168.10.31
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100020 mac-ip 0050.7966.6809
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100020 mac-ip 0050.7966.6809
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65503:100020 mac-ip 0050.7966.6809 192.168.20.31
                                 10.42.204.3           -       100     0       65500 65503 i
 *  ec    RD: 65503:100020 mac-ip 0050.7966.6809 192.168.20.31
                                 10.42.204.3           -       100     0       65500 65503 i
 * >Ec    RD: 65501:100020 mac-ip 0050.7966.680a
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100020 mac-ip 0050.7966.680a
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:100020 mac-ip 0050.7966.680a 192.168.20.11
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100020 mac-ip 0050.7966.680a 192.168.20.11
                                 10.42.204.1           -       100     0       65500 65501 i
 * >      RD: 65502:100020 mac-ip 0050.7966.680b
                                 -                     -       -       0       i
 * >      RD: 65502:100020 mac-ip 0050.7966.680b 192.168.20.21
                                 -                     -       -       0       i
Leaf-2#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.42.204.2
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 100010]      [20, 100020]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4093, 9999]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [EVPN, 9999]
  Headend replication flood vtep list is:
    10 10.42.204.3     10.42.204.1
    20 10.42.204.3     10.42.204.1
  Shared Router MAC is 0000.0000.0000

Leaf-2#sh vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI          VLAN       Source       Interface       802.1Q Tag
------------ ---------- ------------ --------------- ----------
100010       10         static       Ethernet8       untagged
                                     Vxlan1          10
100020       20         static       Ethernet7       untagged
                                     Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI        VLAN       VRF        Source
---------- ---------- ---------- ------------
9999       4093       EVPN       evpn

Leaf-2#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  10.42.204.1      1       0:01:53 ago
  10  0050.7966.6808  EVPN      Vx1  10.42.204.3      1       0:01:38 ago
  20  0050.7966.6809  EVPN      Vx1  10.42.204.3      1       0:01:53 ago
  20  0050.7966.680a  EVPN      Vx1  10.42.204.1      1       0:01:28 ago
4093  5000.00d5.5dc0  EVPN      Vx1  10.42.204.3      1       0:59:40 ago
4093  5000.00d7.ee0b  EVPN      Vx1  10.42.204.1      1       3:08:35 ago
Total Remote Mac Addresses for this criterion: 6
```

- Leaf-3

```
Leaf-3#sh ip route vrf EVPN
 B E      192.168.10.11/32 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.10.21/32 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan110
 B E      192.168.20.11/32 [20/0] via VTEP 10.42.204.1 VNI 9999 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B E      192.168.20.21/32 [20/0] via VTEP 10.42.204.2 VNI 9999 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 C        192.168.20.0/24 is directly connected, Vlan120

Leaf-3#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.42.201.3, local AS number 65503
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65501:100010 mac-ip 0050.7966.6806
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100010 mac-ip 0050.7966.6806
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:100010 mac-ip 0050.7966.6806 192.168.10.11
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100010 mac-ip 0050.7966.6806 192.168.10.11
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65502:100010 mac-ip 0050.7966.6807
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100010 mac-ip 0050.7966.6807
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:100010 mac-ip 0050.7966.6807 192.168.10.21
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100010 mac-ip 0050.7966.6807 192.168.10.21
                                 10.42.204.2           -       100     0       65500 65502 i
 * >      RD: 65503:100010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65503:100010 mac-ip 0050.7966.6808 192.168.10.31
                                 -                     -       -       0       i
 * >      RD: 65503:100020 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 65503:100020 mac-ip 0050.7966.6809 192.168.20.31
                                 -                     -       -       0       i
 * >Ec    RD: 65501:100020 mac-ip 0050.7966.680a
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100020 mac-ip 0050.7966.680a
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65501:100020 mac-ip 0050.7966.680a 192.168.20.11
                                 10.42.204.1           -       100     0       65500 65501 i
 *  ec    RD: 65501:100020 mac-ip 0050.7966.680a 192.168.20.11
                                 10.42.204.1           -       100     0       65500 65501 i
 * >Ec    RD: 65502:100020 mac-ip 0050.7966.680b
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100020 mac-ip 0050.7966.680b
                                 10.42.204.2           -       100     0       65500 65502 i
 * >Ec    RD: 65502:100020 mac-ip 0050.7966.680b 192.168.20.21
                                 10.42.204.2           -       100     0       65500 65502 i
 *  ec    RD: 65502:100020 mac-ip 0050.7966.680b 192.168.20.21
                                 10.42.204.2           -       100     0       65500 65502 i

Leaf-3#sh interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.42.204.3
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [110, 100010]     [120, 100020]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4093, 9999]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [EVPN, 9999]
  Headend replication flood vtep list is:
   110 10.42.204.2     10.42.204.1
   120 10.42.204.2     10.42.204.1
  Shared Router MAC is 0000.0000.0000

Leaf-3#sh vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI          VLAN       Source       Interface       802.1Q Tag
------------ ---------- ------------ --------------- ----------
100010       110        static       Ethernet8       untagged
                                     Vxlan1          110
100020       120        static       Ethernet7       untagged
                                     Vxlan1          120

VNI to dynamic VLAN Mapping for Vxlan1
VNI        VLAN       VRF        Source
---------- ---------- ---------- ------------
9999       4093       EVPN       evpn


Leaf-3#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
 110  0050.7966.6806  EVPN      Vx1  10.42.204.1      1       0:02:03 ago
 110  0050.7966.6807  EVPN      Vx1  10.42.204.2      1       0:01:56 ago
 120  0050.7966.680a  EVPN      Vx1  10.42.204.1      1       0:01:38 ago
 120  0050.7966.680b  EVPN      Vx1  10.42.204.2      1       0:01:31 ago
4093  5000.00cb.38c2  EVPN      Vx1  10.42.204.2      1       1:17:59 ago
4093  5000.00d7.ee0b  EVPN      Vx1  10.42.204.1      1       3:08:44 ago
Total Remote Mac Addresses for this criterion: 6
```

- VPC5
```
VPC5> ping 192.168.10.11 -c 2
84 bytes from 192.168.10.11 icmp_seq=1 ttl=64 time=29.873 ms
84 bytes from 192.168.10.11 icmp_seq=2 ttl=64 time=37.720 ms

VPC5> ping 192.168.10.21 -c 2
84 bytes from 192.168.10.21 icmp_seq=1 ttl=64 time=24.459 ms
84 bytes from 192.168.10.21 icmp_seq=2 ttl=64 time=19.971 ms

VPC5>
VPC5> ping 192.168.20.11 -c 2
84 bytes from 192.168.20.11 icmp_seq=1 ttl=62 time=68.458 ms
84 bytes from 192.168.20.11 icmp_seq=2 ttl=62 time=20.837 ms

VPC5> ping 192.168.20.21 -c 2
84 bytes from 192.168.20.21 icmp_seq=1 ttl=62 time=67.010 ms
84 bytes from 192.168.20.21 icmp_seq=2 ttl=62 time=25.220 ms

VPC5> ping 192.168.20.31 -c 2
84 bytes from 192.168.20.31 icmp_seq=1 ttl=63 time=14.672 ms
84 bytes from 192.168.20.31 icmp_seq=2 ttl=63 time=6.013 ms
```

- VPC6
```
VPC6> ping 192.168.10.11 -c 2
84 bytes from 192.168.10.11 icmp_seq=1 ttl=63 time=294.418 ms
84 bytes from 192.168.10.11 icmp_seq=2 ttl=63 time=23.491 ms

VPC6> ping 192.168.10.21 -c 2
84 bytes from 192.168.10.21 icmp_seq=1 ttl=63 time=127.239 ms
84 bytes from 192.168.10.21 icmp_seq=2 ttl=63 time=43.834 ms

VPC6>
VPC6> ping 192.168.10.31 -c 2
84 bytes from 192.168.10.31 icmp_seq=1 ttl=63 time=76.275 ms
84 bytes from 192.168.10.31 icmp_seq=2 ttl=63 time=6.401 ms

VPC6> ping 192.168.20.11 -c 2
84 bytes from 192.168.20.11 icmp_seq=1 ttl=64 time=59.063 ms
84 bytes from 192.168.20.11 icmp_seq=2 ttl=64 time=13.419 ms

VPC6> ping 192.168.20.21 -c 2
84 bytes from 192.168.20.21 icmp_seq=1 ttl=64 time=15.417 ms
84 bytes from 192.168.20.21 icmp_seq=2 ttl=64 time=17.117 ms

VPC6> ping 192.168.20.31 -c 2
192.168.20.31 icmp_seq=1 ttl=64 time=0.001 ms
192.168.20.31 icmp_seq=2 ttl=64 time=0.001 ms
```
