# Lab 6.0 - SRv6

Consolidação das linhas de configuração do laboratório, reorganizadas por fase e por equipamento.

## Observações

- Este arquivo reúne duas fases do roteiro: `L3VPNv4 over SRv6 BE` e a evolução para `EVPN over SRv6 Policy`.
- As seções abaixo foram separadas para reduzir repetição e facilitar aplicação.
- Comandos de verificação, testes de conectividade e desligamentos temporários de interface foram omitidos.
- O roteiro mostra `undo dcn` apenas como exemplo. Repita a desativação de DCN onde for aplicável no seu ambiente.

## Etapa 1 - L3VPNv4 over SRv6 BE

### PE2

```text
system-view immediately
undo dcn

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::9/126
 quit

interface LoopBack0
 ipv6 enable
 ipv6 address 2001::2:2/128
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0002.0002.0002.00
 is-name PE2
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

ip vpn-instance vpna
 ipv4-family
  route-distinguisher 100:20
  vpn-target 100:1020 both
 quit

interface LoopBack1
 ip binding vpn-instance vpna
 ip address 10.1.2.2 32
 quit

bgp 65100
 router-id 10.0.2.2
 peer 2001::6:6 as-number 65100
 peer 2001::6:6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 2001::6:6 enable
  peer 2001::6:6 prefix-sid
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
  segment-routing ipv6 best-effort
  segment-routing ipv6 locator SRv6
 quit

segment-routing ipv6
 encapsulation source-address 2001::2:2
 locator SRv6 ipv6-prefix 2001:2:: 96 static 16
 quit

isis 1
 segment-routing ipv6 locator SRv6
 quit
```

### P2

```text
system-view immediately

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::a/126
 quit

interface GigabitEthernet3/0/2
 ipv6 enable
 ipv6 address 2001::15/126
 quit

interface LoopBack0
 ipv6 enable
 ipv6 address 2001::6:6/128
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0006.0006.0006.00
 is-name P2
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/2
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

bgp 65100
 router-id 10.0.6.6
 peer 2001::2:2 as-number 65100
 peer 2001::2:2 connect-interface LoopBack0
 peer 2001::4:4 as-number 65100
 peer 2001::4:4 connect-interface LoopBack0
 ipv4-family vpnv4
  undo policy vpn-target
  peer 2001::2:2 enable
  peer 2001::2:2 reflect-client
  peer 2001::4:4 enable
  peer 2001::4:4 reflect-client
  peer 2001::2:2 prefix-sid
  peer 2001::4:4 prefix-sid
 quit
```

### PE4

```text
system-view immediately

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::16/126
 quit

interface LoopBack0
 ipv6 enable
 ipv6 address 2001::4:4/128
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0004.0004.0004.00
 is-name PE4
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

ip vpn-instance vpna
 ipv4-family
  route-distinguisher 100:40
  vpn-target 100:1020 both
 quit

interface LoopBack1
 ip binding vpn-instance vpna
 ip address 10.1.4.4 32
 quit

bgp 65100
 router-id 10.0.4.4
 peer 2001::6:6 as-number 65100
 peer 2001::6:6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 2001::6:6 enable
  peer 2001::6:6 prefix-sid
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
  segment-routing ipv6 best-effort
  segment-routing ipv6 locator SRv6
 quit

segment-routing ipv6
 encapsulation source-address 2001::4:4
 locator SRv6 ipv6-prefix 2001:4:: 96 static 16
 quit

isis 1
 segment-routing ipv6 locator SRv6
 quit
```

## Etapa 2 - EVPN over SRv6 e SRv6 Policy

### PE1

```text
system-view immediately
undo dcn

interface LoopBack0
 ipv6 enable
 ipv6 address 2001::1:1/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::1/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::5/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0001.0001.0001.00
 is-name PE1
 ipv6 enable topology ipv6
 segment-routing ipv6 locator SRv6 auto-sid-disable
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

ip vpn-instance vpna
 ipv4-family
  route-distinguisher 100:10
  vpn-target 100:1020 evpn
  evpn mpls routing-enable
  tnl-policy p1 evpn
  default-color 100 evpn
 quit

interface LoopBack1
 ip binding vpn-instance vpna
 ip address 10.1.1.1 32
 quit

ip vpn-instance vpna6
 ipv6-family
  route-distinguisher 100:11
  vpn-target 100:1122 evpn
  evpn mpls routing-enable
  tnl-policy p1 evpn
  default-color 100 evpn
 quit

interface LoopBack2
 ip binding vpn-instance vpna6
 ipv6 enable
 ipv6 address 2002::1:1 128
 quit

bgp 65100
 router-id 10.0.1.1
 peer 2001::5:5 as-number 65100
 peer 2001::5:5 connect-interface LoopBack0
 peer 2001::6:6 as-number 65100
 peer 2001::6:6 connect-interface LoopBack0
 l2vpn-family evpn
  peer 2001::5:5 enable
  peer 2001::5:5 advertise encap-type srv6
  peer 2001::6:6 enable
  peer 2001::6:6 advertise encap-type srv6
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
  advertise l2vpn evpn
  segment-routing ipv6 locator SRv6 evpn
  segment-routing ipv6 traffic-engineer best-effort evpn
 quit

bgp 65100
 ipv6-family vpn-instance vpna6
  import-route direct
  advertise l2vpn evpn
  segment-routing ipv6 locator SRv6 evpn
  segment-routing ipv6 traffic-engineer best-effort evpn
 quit

segment-routing ipv6
 encapsulation source-address 2001::1:1
 locator SRv6 ipv6-prefix 2001:1:: 96 static 16
  opcode ::1 end
  opcode ::11 end-dt4 vpn-instance vpna evpn
  opcode ::61 end-dt6 vpn-instance vpna6 evpn
 quit

segment-routing ipv6
 segment-list PE1_PE4_VPNA_1
  index 5 sid ipv6 2001:5::1
  index 10 sid ipv6 2001:3::1
  index 15 sid ipv6 2001:4::1
 quit

 segment-list PE1_PE4_VPNA_2
  index 5 sid ipv6 2001:5::1
  index 10 sid ipv6 2001:6::1
  index 15 sid ipv6 2001:4::1
 quit

 segment-list PE1_PE4_VPNA_3
  index 5 sid ipv6 2001:2::1
  index 10 sid ipv6 2001:6::1
  index 15 sid ipv6 2001:4::1
 quit

 srv6-te policy p1 endpoint 2001::4:4 color 100
  candidate-path preference 100
   segment-list PE1_PE4_VPNA_1 weight 2
   segment-list PE1_PE4_VPNA_2 weight 1
  quit
  candidate-path preference 50
   segment-list PE1_PE4_VPNA_3
  quit
 quit

 tunnel-policy p1
  tunnel select-seq ipv6 srv6-te-policy load-balance-number 1
 quit

segment-routing ipv6
 locator SRv6
  opcode ::2 end-op
 quit
 srv6-te-policy path verification enable
 srv6-te policy p1
  path verification enable
 quit
```

### PE2

```text
interface LoopBack0
 ipv6 enable
 ipv6 address 2001::2:2/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::2/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::9/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0002.0002.0002.00
 is-name PE2
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

segment-routing ipv6
 encapsulation source-address 2001::2:2
 locator SRv6 ipv6-prefix 2001:2:: 96 static 16
  opcode ::1 end
 quit
```

### PE3

```text
interface LoopBack0
 ipv6 enable
 ipv6 address 2001::3:3/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::19/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::12/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0003.0003.0003.00
 is-name PE3
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

segment-routing ipv6
 encapsulation source-address 2001::3:3
 locator SRv6 ipv6-prefix 2001:3:: 96 static 16
  opcode ::1 end
 quit
```

### PE4

```text
interface LoopBack0
 ipv6 enable
 ipv6 address 2001::4:4/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::1a/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::16/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0004.0004.0004.00
 is-name PE4
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

ip vpn-instance vpna
 ipv4-family
  route-distinguisher 100:40
  vpn-target 100:1020 evpn
  evpn mpls routing-enable
  tnl-policy p1 evpn
  default-color 100 evpn
 quit

interface LoopBack1
 ip binding vpn-instance vpna
 ip address 10.1.4.4 32
 quit

ip vpn-instance vpna6
 ipv6-family
  route-distinguisher 100:44
  vpn-target 100:1122 evpn
  evpn mpls routing-enable
  tnl-policy p1 evpn
  default-color 100 evpn
 quit

interface LoopBack2
 ip binding vpn-instance vpna6
 ipv6 enable
 ipv6 address 2002::4:4 128
 quit

bgp 65100
 router-id 10.0.4.4
 peer 2001::5:5 as-number 65100
 peer 2001::5:5 connect-interface LoopBack0
 peer 2001::6:6 as-number 65100
 peer 2001::6:6 connect-interface LoopBack0
 l2vpn-family evpn
  peer 2001::5:5 enable
  peer 2001::5:5 advertise encap-type srv6
  peer 2001::6:6 enable
  peer 2001::6:6 advertise encap-type srv6
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
  advertise l2vpn evpn
  segment-routing ipv6 locator SRv6 evpn
  segment-routing ipv6 traffic-engineer best-effort evpn
 quit

bgp 65100
 ipv6-family vpn-instance vpna6
  import-route direct
  advertise l2vpn evpn
  segment-routing ipv6 locator SRv6 evpn
  segment-routing ipv6 traffic-engineer best-effort evpn
 quit

segment-routing ipv6
 encapsulation source-address 2001::4:4
 locator SRv6 ipv6-prefix 2001:4:: 96 static 16
  opcode ::1 end
  opcode ::44 end-dt4 vpn-instance vpna evpn
  opcode ::64 end-dt6 vpn-instance vpna6 evpn
 quit

segment-routing ipv6
 segment-list PE4_PE1_VPNA_1
  index 5 sid ipv6 2001:3::1
  index 10 sid ipv6 2001:5::1
  index 15 sid ipv6 2001:1::1
 quit

 segment-list PE4_PE1_VPNA_2
  index 5 sid ipv6 2001:6::1
  index 10 sid ipv6 2001:5::1
  index 15 sid ipv6 2001:1::1
 quit

 segment-list PE4_PE1_VPNA_3
  index 5 sid ipv6 2001:6::1
  index 10 sid ipv6 2001:2::1
  index 15 sid ipv6 2001:1::1
 quit

 srv6-te policy p1 endpoint 2001::1:1 color 100
  candidate-path preference 100
   segment-list PE4_PE1_VPNA_1 weight 2
   segment-list PE4_PE1_VPNA_2 weight 1
  quit
  candidate-path preference 50
   segment-list PE4_PE1_VPNA_3
  quit
 quit

 tunnel-policy p1
  tunnel select-seq ipv6 srv6-te-policy load-balance-number 1
 quit

segment-routing ipv6
 locator SRv6
  opcode ::2 end-op
 quit
 srv6-te-policy path verification enable
 srv6-te policy p1
  path verification enable
 quit
```

### P1

```text
interface LoopBack0
 ipv6 enable
 ipv6 address 2001::5:5/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::d/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::6/126
 quit

interface GigabitEthernet3/0/2
 ipv6 enable
 ipv6 address 2001::11/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0005.0005.0005.00
 is-name P1
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/2
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

bgp 65100
 router-id 10.0.5.5
 peer 2001::1:1 as-number 65100
 peer 2001::1:1 connect-interface LoopBack0
 peer 2001::4:4 as-number 65100
 peer 2001::4:4 connect-interface LoopBack0
 l2vpn-family evpn
  undo policy vpn-target
  peer 2001::1:1 enable
  peer 2001::1:1 advertise encap-type srv6
  peer 2001::1:1 reflect-client
  peer 2001::4:4 enable
  peer 2001::4:4 advertise encap-type srv6
  peer 2001::4:4 reflect-client
 quit

segment-routing ipv6
 encapsulation source-address 2001::5:5
 locator SRv6 ipv6-prefix 2001:5:: 96 static 16
  opcode ::1 end
 quit
```

### P2

```text
interface LoopBack0
 ipv6 enable
 ipv6 address 2001::6:6/128
 quit

interface GigabitEthernet3/0/0
 ipv6 enable
 ipv6 address 2001::e/126
 quit

interface GigabitEthernet3/0/1
 ipv6 enable
 ipv6 address 2001::a/126
 quit

interface GigabitEthernet3/0/2
 ipv6 enable
 ipv6 address 2001::15/126
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0006.0006.0006.00
 is-name P2
 ipv6 enable topology ipv6
 quit

interface GigabitEthernet3/0/0
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/2
 isis ipv6 enable 1
 isis circuit-type p2p
 quit

interface LoopBack0
 isis ipv6 enable 1
 quit

bgp 65100
 router-id 10.0.6.6
 peer 2001::1:1 as-number 65100
 peer 2001::1:1 connect-interface LoopBack0
 peer 2001::4:4 as-number 65100
 peer 2001::4:4 connect-interface LoopBack0
 l2vpn-family evpn
  undo policy vpn-target
  peer 2001::1:1 enable
  peer 2001::1:1 advertise encap-type srv6
  peer 2001::1:1 reflect-client
  peer 2001::4:4 enable
  peer 2001::4:4 advertise encap-type srv6
  peer 2001::4:4 reflect-client
 quit

segment-routing ipv6
 encapsulation source-address 2001::6:6
 locator SRv6 ipv6-prefix 2001:6:: 96 static 16
  opcode ::1 end
 quit
```
