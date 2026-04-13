# Lab 3.0 - EVPN

Consolidação das linhas de configuração do laboratório, agrupadas por equipamento.

## Observações

- O conteúdo abaixo foi consolidado a partir do roteiro do laboratório.
- Comandos de verificação, testes de conectividade, capturas de pacote e desligamentos temporários de interface foram omitidos.
- Em laboratórios incrementais, alguns blocos podem reaparecer porque o roteiro recompõe a configuração base em etapas posteriores.
- Quando o roteiro mostra `undo dcn` em apenas um equipamento como exemplo, repita o comando nos demais equipamentos Huawei aplicáveis.

## PE1

```text
system-view immediately

undo dcn

interface LoopBack0
 ip address 10.0.1.1 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.1 30
quit

ospf 1 router-id 10.0.1.1
 area 0.0.0.0

interface GigabitEthernet3/5/0

ospf enable 1 area 0.0.0.0

interface LoopBack1
 ip address 10.1.1.1 32

ospf enable 1 area 0.0.0.0
```

## PE2

```text
system-view immediately

interface LoopBack0
 ip address 10.0.2.2 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.2 30
quit

interface GigabitEthernet3/5/1
 ip address 10.0.0.9 30
quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0002.0002.0002.00
 is-name PE2

interface LoopBack0

isis enable 1
quit

interface GigabitEthernet3/5/1

isis enable 1

isis circuit-type p2p
quit

mpls lsr-id 10.0.2.2

mpls
 mpls ldp

interface GigabitEthernet3/5/1

mpls
 mpls ldp

bgp 65100
 router-id 10.0.2.2
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 l2vpn-family evpn
 policy vpn-target
 peer 10.0.6.6 enable

ip vpn-instance vpna
 ipv4-family
 route-distinguisher 10:20
 apply-label per-instance
 vpn-target 10:26 export-extcommunity evpn
 vpn-target 10:624 import-extcommunity evpn
 evpn mpls routing-enable

interface GigabitEthernet3/5/0
 ip binding vpn-instance vpna
 ip address 10.0.0.2 255.255.255.252

ospf 1 router-id 10.0.2.2 vpn-instance vpna
 import-route bgp cost 20 type 2
 area 0.0.0.0

interface GigabitEthernet3/5/0

ospf enable 1 area 0.0.0.0

ip ip-prefix Loopback1 index 10 permit 10.1.1.1 32
 route-policy O2B permit node 10
 if-match ip-prefix Loopback1

bgp 65100
 ipv4-family vpn-instance vpna
 import-route ospf 1 route-policy O2B
 advertise l2vpn evpn
```

## PE3

```text
system-view immediately

interface LoopBack0
 ip address 10.0.3.3 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.25 30
quit

bgp 65003
 router-id 10.0.3.3
 peer 10.0.0.26 as-number 65100

interface LoopBack1
 ip address 10.1.3.3 32
quit

bgp 65003
 network 10.1.3.3 255.255.255.255
```

## PE4

```text
system-view immediately

interface LoopBack0
 ip address 10.0.4.4 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.26 30
quit

interface GigabitEthernet3/5/1
 undo shutdown
 ip address 10.0.0.22 255.255.255.252
quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0004.0004.0004.00
 is-name PE4
quit

interface LoopBack0

isis enable 1
quit

interface GigabitEthernet3/5/1

isis enable 1

isis circuit-type p2p
quit

mpls lsr-id 10.0.4.4

mpls
 mpls ldp

interface GigabitEthernet3/5/1

mpls
 mpls ldp

bgp 65100
 router-id 10.0.4.4
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 l2vpn-family evpn
 policy vpn-target
 peer 10.0.6.6 enable

ip vpn-instance vpna
 ipv4-family
 route-distinguisher 10:40
 vpn-target 10:46 export-extcommunity evpn
 vpn-target 10:624 import-extcommunity evpn
 evpn mpls routing-enable
quit

interface GigabitEthernet3/5/0
 ip binding vpn-instance vpna
 ip address 10.0.0.26 255.255.255.252

bgp 65100
 ipv4-family vpn-instance vpna
 peer 10.0.0.25 as-number 65003
 peer 10.0.0.25 substitute-as
 advertise l2vpn evpn
```

## P1

```text
system-view immediately

interface LoopBack0
 ip address 10.0.5.5 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.13 30
quit

interface GigabitEthernet3/5/0.1

vlan-type dot1q 10
 ip address 10.0.0.113 255.255.255.252
quit

bgp 65003
 router-id 10.0.5.5
 peer 10.0.0.14 as-number 65100
 peer 10.0.0.114 as-number 65100

interface LoopBack1
 ip address 10.1.5.5 32

bgp 65003
 network 10.1.5.5 255.255.255.255
```

## P2

```text
system-view immediately

interface LoopBack0
 ip address 10.0.6.6 32
quit

interface GigabitEthernet3/5/0
 ip address 10.0.0.14 30
quit

interface GigabitEthernet3/5/1
 ip address 10.0.0.10 30
quit

interface GigabitEthernet3/5/2
 undo shutdown
 ip address 10.0.0.21 255.255.255.252

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0006.0006.0006.00
 is-name P2
quit

interface LoopBack0

isis enable 1
quit

interface GigabitEthernet3/5/1

isis enable 1

isis circuit-type p2p
quit

interface GigabitEthernet3/5/2

isis enable 1

isis circuit-type p2p
quit

mpls lsr-id 10.0.6.6

mpls
 mpls ldp

interface GigabitEthernet3/5/1

mpls
 mpls ldp

interface GigabitEthernet3/5/2

mpls
 mpls ldp
quit

bgp 65100
 router-id 10.0.6.6
 peer 10.0.2.2 as-number 65100
 peer 10.0.2.2 connect-interface LoopBack0
 peer 10.0.4.4 as-number 65100
 peer 10.0.4.4 connect-interface LoopBack0
 l2vpn-family evpn
 undo policy vpn-target
 peer 10.0.2.2 enable
 peer 10.0.2.2 reflect-client
 peer 10.0.4.4 enable
 peer 10.0.4.4 reflect-client

interface GigabitEthernet3/5/0.1

vlan-type dot1q 10
 ip address 10.0.0.114 255.255.255.252
quit

ip vpn-instance vpna_in
 ipv4-family
 route-distinguisher 10:60
 vpn-target 10:46 import-extcommunity evpn
 vpn-target 10:26 import-extcommunity evpn
 evpn mpls routing-enable
quit

ip vpn-instance vpna_out
 ipv4-family
 route-distinguisher 10:6060
 vpn-target 10:624 export-extcommunity evpn
 evpn mpls routing-enable
quit

interface GigabitEthernet3/5/0
 ip binding vpn-instance vpna_out
 ip address 10.0.0.14 255.255.255.252
quit

interface GigabitEthernet3/5/0.1

vlan-type dot1q 10
 ip binding vpn-instance vpna_in
 ip address 10.0.0.114 255.255.255.252

bgp 65100
 ipv4-family vpn-instance vpna_in
 peer 10.0.0.113 as-number 65003
 peer 10.0.0.113 substitute-as
 advertise l2vpn evpn
 ipv4-family vpn-instance vpna_out
 peer 10.0.0.13 as-number 65003
 peer 10.0.0.13 allow-as-loop 2
 advertise l2vpn evpn
```


