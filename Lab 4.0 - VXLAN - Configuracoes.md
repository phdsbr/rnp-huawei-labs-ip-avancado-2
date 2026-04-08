# Lab 4.0 - VXLAN

Consolidação das linhas de configuração do laboratório, agrupadas por equipamento.

## Observações

- O conteúdo abaixo foi consolidado a partir do roteiro do laboratório.
- Comandos de verificação, testes de conectividade, capturas de pacote e desligamentos temporários de interface foram omitidos.
- Em laboratórios incrementais, alguns blocos podem reaparecer porque o roteiro recompõe a configuração base em etapas posteriores.
- Quando o roteiro mostra `undo dcn` em apenas um equipamento como exemplo, repita o comando nos demais equipamentos Huawei aplicáveis.

## Edge_1

```text
vlan 12
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 12
quit
interface Vlanif12
ip address 10.0.12.1 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.1.1 32
quit
ospf 1 router-id 10.0.1.1
area 0.0.0.0
network 10.0.1.1 0.0.0.0
network 10.0.12.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.100 mode l2
encapsulation dot1q vid 100
bridge-domain 10
quit
interface Nve1
source 10.0.1.1
vni 1000 head-end peer-list 10.0.3.3
quit
vlan 12
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 12
quit
interface Vlanif12
ip address 10.0.12.1 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.1.1 32
quit
ospf 1 router-id 10.0.1.1
area 0.0.0.0
network 10.0.1.1 0.0.0.0
network 10.0.12.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.100 mode l2
encapsulation dot1q vid 100
bridge-domain 10
quit
interface Nve1
source 10.0.1.1
vni 1000 head-end peer-list 10.0.2.2
quit
vlan 12
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 12
quit
interface Vlanif12
ip address 10.0.12.1 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.1.1 32
quit
ospf 1 router-id 10.0.1.1
area 0.0.0.0
network 10.0.1.1 0.0.0.0
network 10.0.12.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
bridge-domain 20
vxlan vni 2000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.100 mode l2
encapsulation dot1q vid 100
bridge-domain 10
quit
interface Nve1
source 10.0.1.1
vni 1000 head-end peer-list protocol bgp
vni 2000 head-end peer-list protocol bgp
quit
interface Vbdif10
ip address 192.168.100.254 255.255.255.0
mac-address 0000-5e00-0110
arp collect host enable
quit
interface Vbdif20
ip address 192.168.200.254 255.255.255.0
mac-address 0000-5e00-0120
arp collect host enable
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
bridge-domain 10
evpn binding vpn-instance Edge_1_BD_10
quit
bridge-domain 20
evpn binding vpn-instance Edge_1_BD_20
quit
ip vpn-instance Edge_1_VPN_10_20
ipv4-family
route-distinguisher 31:12
vpn-target 313:12 export-extcommunity evpn
vpn-target 313:12 import-extcommunity evpn
quit
vxlan vni 1020
quit
interface Vbdif 10
ip binding vpn-instance Edge_1_VPN_10_20
quit
interface Vbdif 20
ip binding vpn-instance Edge_1_VPN_10_20
quit
interface Vbdif10
ip address 192.168.100.254 255.255.255.0
quit
interface Vbdif20
ip address 192.168.200.254 255.255.255.0
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
```

## Border

```text
vlan batch 12 23
interface GigabitEthernet0/0/23
port link-type trunk
port trunk allow-pass vlan 12
quit
interface GigabitEthernet0/0/24
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif12
ip address 10.0.12.2 255.255.255.0
quit
interface Vlanif23
ip address 10.0.23.2 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.2.2 32
quit
ospf 1 router-id 10.0.2.2
area 0.0.0.0
network 10.0.2.2 0.0.0.0
network 10.0.12.0 0.0.0.255
network 10.0.23.0 0.0.0.255
quit
acl number 3000
rule 1 permit udp source 10.0.1.1 0 destination 10.0.3.3 0 destination-port eq 4789
vlan batch 12 23
interface GigabitEthernet0/0/23
port link-type trunk
port trunk allow-pass vlan 12
quit
interface GigabitEthernet0/0/24
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif12
ip address 10.0.12.2 255.255.255.0
quit
interface Vlanif23
ip address 10.0.23.2 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.2.2 32
quit
ospf 1 router-id 10.0.2.2
area 0.0.0.0
network 10.0.2.2 0.0.0.0
network 10.0.12.0 0.0.0.255
network 10.0.23.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
interface Nve1
source 10.0.2.2
vni 1000 head-end peer-list 10.0.1.1
quit
bridge-domain 20
vxlan vni 2000
quit
interface Nve 1
vni 2000 head-end peer-list 10.0.3.3
quit
interface Vbdif10
ip address 192.168.100.254 255.255.255.0
quit
interface Vbdif20
ip address 192.168.200.254 255.255.255.0
quit
vlan batch 12 23
interface GigabitEthernet0/0/23
port link-type trunk
port trunk allow-pass vlan 12
quit
interface GigabitEthernet0/0/24
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif12
ip address 10.0.12.2 255.255.255.0
quit
interface Vlanif23
ip address 10.0.23.2 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.2.2 32
quit
ospf 1 router-id 10.0.2.2
area 0.0.0.0
network 10.0.2.2 0.0.0.0
network 10.0.12.0 0.0.0.255
network 10.0.23.0 0.0.0.255
quit
```

## Edge_2

```text
vlan 23
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif 23
ip address 10.0.23.3 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.3.3 32
quit
ospf 1 router-id 10.0.3.3
area 0.0.0.0
network 10.0.3.3 0.0.0.0
network 10.0.23.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.100 mode l2
encapsulation dot1q vid 100
bridge-domain 10
quit
interface Nve1
source 10.0.3.3
vni 1000 head-end peer-list 10.0.1.1
quit
vlan 23
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif 23
ip address 10.0.23.3 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.3.3 32
quit
ospf 1 router-id 10.0.3.3
area 0.0.0.0
network 10.0.3.3 0.0.0.0
network 10.0.23.0 0.0.0.255
quit
bridge-domain 20
vxlan vni 2000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.200 mode l2
encapsulation dot1q vid 200
bridge-domain 20
quit
interface Nve1
source 10.0.3.3
vni 2000 head-end peer-list 10.0.2.2
quit
vlan 23
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 23
quit
interface Vlanif 23
ip address 10.0.23.3 255.255.255.0
quit
interface LoopBack 0
ip address 10.0.3.3 32
quit
ospf 1 router-id 10.0.3.3
area 0.0.0.0
network 10.0.3.3 0.0.0.0
network 10.0.23.0 0.0.0.255
quit
bridge-domain 10
vxlan vni 1000
quit
bridge-domain 20
vxlan vni 2000
quit
vcmp role silent
interface GigabitEthernet0/0/24
port link-type trunk
quit
interface GigabitEthernet0/0/24.200 mode l2
encapsulation dot1q vid 200
bridge-domain 20
quit
interface GigabitEthernet0/0/24.100 mode l2
encapsulation dot1q vid 100
bridge-domain 10
quit
interface Nve1
source 10.0.3.3
vni 1000 head-end peer-list protocol bgp
vni 2000 head-end peer-list protocol bgp
quit
interface Vbdif10
ip address 192.168.100.254 255.255.255.0
mac-address 0000-5e00-0110
arp collect host enable
quit
interface Vbdif20
ip address 192.168.200.254 255.255.255.0
mac-address 0000-5e00-0120
arp collect host enable
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
bridge-domain 10
evpn binding vpn-instance Edge_2_BD_10
quit
bridge-domain 20
evpn binding vpn-instance Edge_2_BD_20
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
interface Vbdif 10
ip binding vpn-instance Edge_2_VPN_10_20
quit
interface Vbdif 20
ip binding vpn-instance Edge_2_VPN_10_20
quit
interface Vbdif10
ip address 192.168.100.254 255.255.255.0
quit
interface Vbdif20
ip address 192.168.200.254 255.255.255.0
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
```

## ACC_1

```text
vlan 100
interface GigabitEthernet0/0/24
port link-type access
port default vlan 100
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 100
quit
vlan 100
interface GigabitEthernet0/0/24
port link-type access
port default vlan 100
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 100
quit
vlan batch 100 200
interface GigabitEthernet0/0/24
port link-type access
port default vlan 100
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 100 200
quit
```

## ACC_2

```text
vlan 100
interface GigabitEthernet0/0/23
port link-type access
port default vlan 100
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 100
quit
vlan 200
interface GigabitEthernet0/0/23
port link-type access
port default vlan 200
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 200
quit
vlan batch 100 200
interface GigabitEthernet0/0/23
port link-type access
port default vlan 200
quit
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 100 200
quit
interface vlanif100
ip address 192.168.100.2 255.255.255.0
quit
ip route-static 192.168.200.0 255.255.255.0 192.168.100.254
```


