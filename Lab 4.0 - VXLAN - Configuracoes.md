# Lab 4.0 - VXLAN

Consolidação das configurações do laboratório, sem repetições, organizada por equipamento e por etapa do roteiro.

## Topologia

![Topologia do Lab 4.0 - VXLAN](./Lab%204.0%20-%20VXLAN%20-%20Topologia.png)

## Como usar este arquivo

- `Base comum`: aplicar em qualquer uma das etapas do Lab 4.0.
- `Etapa 1`: VXLAN estático para extensão L2 entre `Edge_1` e `Edge_2`.
- `Etapa 2`: gateway VXLAN centralizado com `Border` fazendo o papel de gateway.
- `Etapa 3`: gateway VXLAN distribuído com EVPN/IRB entre `Edge_1` e `Edge_2`.
- Para a `Etapa 3`, consulte também a imagem ![`Lab 4.0 - VXLAN - Topologia - EVPN.png`](./Lab%204.0%20-%20VXLAN%20-%20Topologia%20-%20EVPN.png).
- As etapas abaixo foram tratadas como cenários independentes do roteiro original. Por isso, os blocos específicos de cada etapa devem ser aplicados sobre a `Base comum`, e não uns sobre os outros.
- Comandos de verificação, `ping`, `display`, `capture-packet`, ACL temporária para captura e descrições explicativas do roteiro foram omitidos.

## Edge_1

### Base comum

```text
vlan 12

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 12
quit

interface Vlanif12
 ip address 10.0.12.1 255.255.255.0
quit

interface LoopBack0
 ip address 10.0.1.1 255.255.255.255
quit

ospf 1 router-id 10.0.1.1
 area 0.0.0.0
  network 10.0.1.1 0.0.0.0
  network 10.0.12.0 0.0.0.255
quit
```

### Etapa 1 - VXLAN estático L2

```text
bridge-domain 10
 vxlan vni 1000
quit

vcmp role silent

interface GE1/0/10
 port link-type trunk
quit

interface GE1/0/10.100 mode l2
 encapsulation dot1q vid 100
 bridge-domain 10
quit

interface Nve1
 source 10.0.1.1
 vni 1000 head-end peer-list 10.0.3.3
quit
```

### Etapa 2 - Gateway centralizado

```text
bridge-domain 10
 vxlan vni 1000
quit

vcmp role silent

interface GE1/0/10
 port link-type trunk
quit

interface GE1/0/10.100 mode l2
 encapsulation dot1q vid 100
 bridge-domain 10
quit

interface Nve1
 source 10.0.1.1
 vni 1000 head-end peer-list 10.0.2.2
quit
```

### Etapa 3 - Gateway distribuído com EVPN

```text
vlan 12

interface GigabitEthernet3/0/1
 portswitch
 port link-type trunk
 port trunk allow-pass vlan 12
quit

interface Vlanif12
 ip address 10.0.12.1 255.255.255.0
quit

interface LoopBack0
 ip address 10.0.1.1 255.255.255.255
quit

ospf 1 router-id 10.0.1.1
 area 0.0.0.0
  network 10.0.1.1 0.0.0.0
  network 10.0.12.0 0.0.0.255
quit

bridge-domain 10
 vxlan vni 1000
 evpn binding vpn-instance Edge_1_BD_10
quit

bridge-domain 20
 vxlan vni 2000
 evpn binding vpn-instance Edge_1_BD_20
quit

interface GigabitEthernet3/0/9
 portswitch
 port link-type trunk
quit
#
interface GigabitEthernet3/0/9.100 mode l2
 encapsulation dot1q vid 100
 rewrite pop single
 bridge-domain 10
#
interface Vbdif10                          
 ip address 10.10.10.1 255.255.255.0
#

interface Nve1
 source 10.0.1.1
 vni 1000 head-end peer-list protocol bgp
 vni 2000 head-end peer-list protocol bgp
quit

interface Vbdif10
 mac-address 0000-5e00-0110
 arp collect host enable
 ip binding vpn-instance Edge_1_VPN_10_20
 ip address 192.168.100.254 255.255.255.0
quit

interface Vbdif20
 mac-address 0000-5e00-0120
 arp collect host enable
 ip binding vpn-instance Edge_1_VPN_10_20
 ip address 192.168.200.254 255.255.255.0
quit

evpn vpn-instance Edge_1_BD_10 bd-mode
 route-distinguisher 21:10
 vpn-target 213:10 export-extcommunity
 vpn-target 313:12 export-extcommunity
 vpn-target 213:10 import-extcommunity
quit

evpn vpn-instance Edge_1_BD_20 bd-mode
 route-distinguisher 21:20
 vpn-target 213:20 export-extcommunity
 vpn-target 313:12 export-extcommunity
 vpn-target 213:20 import-extcommunity
quit

ip vpn-instance Edge_1_VPN_10_20
 ipv4-family
  route-distinguisher 31:12
  vpn-target 313:12 export-extcommunity evpn
  vpn-target 313:12 import-extcommunity evpn
 quit
 vxlan vni 1020
quit

bgp 100
 router-id 10.0.1.1
 peer 10.0.3.3 as-number 100
 peer 10.0.3.3 connect-interface LoopBack0

 ipv4-family vpn-instance Edge_1_VPN_10_20
  advertise l2vpn evpn
 quit

 l2vpn-family evpn
  policy vpn-target
  peer 10.0.3.3 enable
  peer 10.0.3.3 advertise irb
 quit
quit
```

## Border

### Base comum

```text
vlan batch 12 23

interface GE1/0/9
 port link-type trunk
 port trunk allow-pass vlan 12
quit

interface GE1/0/10
 port link-type trunk
 port trunk allow-pass vlan 23
quit

interface Vlanif12
 ip address 10.0.12.2 255.255.255.0
quit

interface Vlanif23
 ip address 10.0.23.2 255.255.255.0
quit

interface LoopBack0
 ip address 10.0.2.2 255.255.255.255
quit

ospf 1 router-id 10.0.2.2
 area 0.0.0.0
  network 10.0.2.2 0.0.0.0
  network 10.0.12.0 0.0.0.255
  network 10.0.23.0 0.0.0.255
quit
```

### Etapa 1 - VXLAN estático L2

Sem configuração adicional no `Border`.

### Etapa 2 - Gateway centralizado

```text
bridge-domain 10
 vxlan vni 1000
quit

bridge-domain 20
 vxlan vni 2000
quit

interface Nve1
 source 10.0.2.2
 vni 1000 head-end peer-list 10.0.1.1
 vni 2000 head-end peer-list 10.0.3.3
quit

interface Vbdif10
 ip address 192.168.100.254 255.255.255.0
quit

interface Vbdif20
 ip address 192.168.200.254 255.255.255.0
quit
```

### Etapa 3 - Gateway distribuído com EVPN

Sem configuração adicional no `Border` além da `Base comum`.

## Edge_2

### Base comum

```text
vlan 23

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 23
quit

interface Vlanif23
 ip address 10.0.23.3 255.255.255.0
quit

interface LoopBack0
 ip address 10.0.3.3 255.255.255.255
quit

ospf 1 router-id 10.0.3.3
 area 0.0.0.0
  network 10.0.3.3 0.0.0.0
  network 10.0.23.0 0.0.0.255
quit
```

### Etapa 1 - VXLAN estático L2

```text
bridge-domain 10
 vxlan vni 1000
quit

interface GE1/0/10
 port link-type trunk
quit

interface GE1/0/10.100 mode l2
 encapsulation dot1q vid 100
 bridge-domain 10
quit

interface Nve1
 source 10.0.3.3
 vni 1000 head-end peer-list 10.0.1.1
quit
```

### Etapa 2 - Gateway centralizado

```text
bridge-domain 20
 vxlan vni 2000
quit

interface GE1/0/10
 port link-type trunk
quit

interface GE1/0/10.200 mode l2
 encapsulation dot1q vid 200
 bridge-domain 20
quit

interface Nve1
 source 10.0.3.3
 vni 2000 head-end peer-list 10.0.2.2
quit
```

### Etapa 3 - Gateway distribuído com EVPN

```text
vlan 23

interface GigabitEthernet3/0/1
 portswitch
 port link-type trunk
 port trunk allow-pass vlan 23
quit

interface Vlanif23
 ip address 10.0.23.3 255.255.255.0
quit

interface LoopBack0
 ip address 10.0.3.3 255.255.255.255
quit

ospf 1 router-id 10.0.3.3
 area 0.0.0.0
  network 10.0.3.3 0.0.0.0
  network 10.0.23.0 0.0.0.255
quit

bridge-domain 10
 vxlan vni 1000
 evpn binding vpn-instance Edge_2_BD_10
quit

bridge-domain 20
 vxlan vni 2000
 evpn binding vpn-instance Edge_2_BD_20
quit

vcmp role silent

interface GigabitEthernet3/0/9
 portswitch
 port link-type trunk
quit

interface GigabitEthernet3/0/9.100 mode l2
 encapsulation dot1q vid 100
 rewrite pop single
 bridge-domain 10
quit

interface GigabitEthernet3/0/9.200 mode l2
 encapsulation dot1q vid 200
 rewrite pop single
 bridge-domain 20
quit

interface Nve1
 source 10.0.3.3
 vni 1000 head-end peer-list protocol bgp
 vni 2000 head-end peer-list protocol bgp
quit

interface Vbdif10
 mac-address 0000-5e00-0110
 arp collect host enable
 ip binding vpn-instance Edge_2_VPN_10_20
 ip address 192.168.100.254 255.255.255.0
quit

interface Vbdif20
 mac-address 0000-5e00-0120
 arp collect host enable
 ip binding vpn-instance Edge_2_VPN_10_20
 ip address 192.168.200.254 255.255.255.0
quit

evpn vpn-instance Edge_2_BD_10 bd-mode
 route-distinguisher 23:10
 vpn-target 213:10 export-extcommunity
 vpn-target 313:12 export-extcommunity
 vpn-target 213:10 import-extcommunity
quit

evpn vpn-instance Edge_2_BD_20 bd-mode
 route-distinguisher 23:20
 vpn-target 213:20 export-extcommunity
 vpn-target 313:12 export-extcommunity
 vpn-target 213:20 import-extcommunity
quit

ip vpn-instance Edge_2_VPN_10_20
 ipv4-family
  route-distinguisher 33:12
  vpn-target 313:12 export-extcommunity
  vpn-target 313:12 export-extcommunity evpn
  vpn-target 313:12 import-extcommunity
  vpn-target 313:12 import-extcommunity evpn
 quit
 vxlan vni 1020
quit

bgp 100
 router-id 10.0.3.3
 peer 10.0.1.1 as-number 100
 peer 10.0.1.1 connect-interface LoopBack0

 ipv4-family vpn-instance Edge_2_VPN_10_20
  advertise l2vpn evpn
 quit

 l2vpn-family evpn
  policy vpn-target
  peer 10.0.1.1 enable
  peer 10.0.1.1 advertise irb
 quit
quit
```

## ACC_1

### Etapas 1 e 2

```text
vlan 100

interface GE1/0/10
 port link-type access
 port default vlan 100
quit

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 100
quit
```

### Etapa 3 - Gateway distribuído com EVPN

```text
vlan batch 100 200

interface GE1/0/10
 port link-type access
 port default vlan 100
quit

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 100 200
quit
```

## ACC_2

### Etapa 1 - VXLAN estático L2

```text
vlan 100

interface GE1/0/9
 port link-type access
 port default vlan 100
quit

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 100
quit
```

### Etapa 2 - Gateway centralizado

```text
vlan 200

interface GE1/0/9
 port link-type access
 port default vlan 200
quit

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 200
quit
```

### Etapa 3 - Gateway distribuído com EVPN

```text
vlan batch 100 200

interface GE1/0/9
 port link-type access
 port default vlan 200
quit

interface GE1/0/1
 port link-type trunk
 port trunk allow-pass vlan 100 200
quit

interface Vlanif100
 ip address 192.168.100.2 255.255.255.0
quit

ip route-static 192.168.200.0 255.255.255.0 192.168.100.254
```


