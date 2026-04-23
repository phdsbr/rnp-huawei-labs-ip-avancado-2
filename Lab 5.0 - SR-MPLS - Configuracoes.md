# Lab 5.0 - SR-MPLS

Consolidação das linhas de configuração do laboratório, reorganizadas por etapa e por equipamento.

## Observações

- Este laboratório é incremental. As seções abaixo separam a configuração base da evolução para `explicit-path`, `SR Policy` e proteção.
- Comandos de verificação, testes de conectividade e desligamentos temporários de interface foram omitidos.
- O roteiro mostra `undo dcn` apenas como exemplo em um dos equipamentos. Repita a desativação de DCN onde for aplicável no seu ambiente.

## Etapa 1 - Base L3VPNv4 over SR-MPLS BE

### PE1

```text
system-view immediately
undo dcn

interface LoopBack0
 ip address 10.0.1.1 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.1 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.5 30
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0001.0001.0001.00
 is-name PE1
 quit

interface LoopBack0
 isis enable 1
 quit

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.1.1
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 1
 quit

ip vpn-instance vpna
 ipv4-family
  route-distinguisher 100:10
  vpn-target 100:1020 both
 quit

interface LoopBack1
 ip binding vpn-instance vpna
 ip address 10.1.1.1 32
 quit

bgp 65100
 router-id 10.0.1.1
 peer 10.0.5.5 as-number 65100
 peer 10.0.5.5 connect-interface LoopBack0
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 10.0.5.5 enable
  peer 10.0.6.6 enable
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
 quit
```

### PE2

```text
system-view immediately

interface LoopBack0
 ip address 10.0.2.2 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.2 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.9 30
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0002.0002.0002.00
 is-name PE2
 quit

interface LoopBack0
 isis enable 1
 quit

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.2.2
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 2
 quit

bgp 65100
 router-id 10.0.2.2
 peer 10.0.5.5 as-number 65100
 peer 10.0.5.5 connect-interface LoopBack0
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 10.0.5.5 enable
  peer 10.0.6.6 enable
 quit
```

### PE3

```text
system-view immediately

interface LoopBack0
 ip address 10.0.3.3 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.25 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.18 30
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0003.0003.0003.00
 is-name PE3
 quit

interface LoopBack0
 isis enable 1
 quit

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.3.3
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 3
 quit

bgp 65100
 router-id 10.0.3.3
 peer 10.0.5.5 as-number 65100
 peer 10.0.5.5 connect-interface LoopBack0
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 10.0.5.5 enable
  peer 10.0.6.6 enable
 quit
```

### PE4

```text
system-view immediately

interface LoopBack0
 ip address 10.0.4.4 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.26 30
 quit

interface GigabitEthernet3/0/1
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

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.4.4
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 4
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
 peer 10.0.5.5 as-number 65100
 peer 10.0.5.5 connect-interface LoopBack0
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 ipv4-family vpnv4
  peer 10.0.5.5 enable
  peer 10.0.6.6 enable
 quit

bgp 65100
 ipv4-family vpn-instance vpna
  import-route direct
 quit
```

### P1

```text
system-view immediately

interface LoopBack0
 ip address 10.0.5.5 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.13 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.6 30
 quit

interface GigabitEthernet3/0/2
 ip address 10.0.0.17 30
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0005.0005.0005.00
 is-name P1
 quit

interface LoopBack0
 isis enable 1
 quit

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/2
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.5.5
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 5
 quit

bgp 65100
 router-id 10.0.5.5
 peer 10.0.1.1 as-number 65100
 peer 10.0.1.1 connect-interface LoopBack0
 peer 10.0.2.2 as-number 65100
 peer 10.0.2.2 connect-interface LoopBack0
 peer 10.0.3.3 as-number 65100
 peer 10.0.3.3 connect-interface LoopBack0
 peer 10.0.4.4 as-number 65100
 peer 10.0.4.4 connect-interface LoopBack0
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.0.1.1 enable
  peer 10.0.1.1 reflect-client
  peer 10.0.2.2 enable
  peer 10.0.2.2 reflect-client
  peer 10.0.3.3 enable
  peer 10.0.3.3 reflect-client
  peer 10.0.4.4 enable
  peer 10.0.4.4 reflect-client
 quit
```

### P2

```text
system-view immediately

interface LoopBack0
 ip address 10.0.6.6 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.14 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.10 30
 quit

interface GigabitEthernet3/0/2
 undo shutdown
 ip address 10.0.0.21 255.255.255.252
 quit

isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0001.0006.0006.0006.00
 is-name P2
 quit

interface LoopBack0
 isis enable 1
 quit

interface GigabitEthernet3/0/0
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/1
 isis enable 1
 isis circuit-type p2p
 quit

interface GigabitEthernet3/0/2
 isis enable 1
 isis circuit-type p2p
 quit

mpls lsr-id 10.0.6.6
mpls

segment-routing
 quit

isis 1
 segment-routing mpls
 segment-routing global-block 16000 17000
 quit

interface LoopBack0
 isis prefix-sid index 6
 quit

bgp 65100
 router-id 10.0.6.6
 peer 10.0.1.1 as-number 65100
 peer 10.0.1.1 connect-interface LoopBack0
 peer 10.0.2.2 as-number 65100
 peer 10.0.2.2 connect-interface LoopBack0
 peer 10.0.3.3 as-number 65100
 peer 10.0.3.3 connect-interface LoopBack0
 peer 10.0.4.4 as-number 65100
 peer 10.0.4.4 connect-interface LoopBack0
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.0.1.1 enable
  peer 10.0.1.1 reflect-client
  peer 10.0.2.2 enable
  peer 10.0.2.2 reflect-client
  peer 10.0.3.3 enable
  peer 10.0.3.3 reflect-client
  peer 10.0.4.4 enable
  peer 10.0.4.4 reflect-client
 quit
```

## Etapa 2 - Adj-SIDs e Túneis SR-TE por Explicit Path

### P1

```text
segment-routing
 ipv4 adjacency local-ip-addr 10.0.0.6 remote-ip-addr 10.0.0.5 sid 142336
 ipv4 adjacency local-ip-addr 10.0.0.13 remote-ip-addr 10.0.0.14 sid 142337
 ipv4 adjacency local-ip-addr 10.0.0.17 remote-ip-addr 10.0.0.18 sid 142338
 quit
```

### P2

```text
segment-routing
 ipv4 adjacency local-ip-addr 10.0.0.10 remote-ip-addr 10.0.0.9 sid 142336
 ipv4 adjacency local-ip-addr 10.0.0.14 remote-ip-addr 10.0.0.13 sid 142337
 ipv4 adjacency local-ip-addr 10.0.0.21 remote-ip-addr 10.0.0.22 sid 142338
 quit
```

### PE1

```text
explicit-path PE1_PE4_Manual
 next sid label 16005 type prefix
 next sid label 142337 type adjacency
 next sid label 142338 type adjacency
 quit

mpls
 mpls te
 quit

interface Tunnel10
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 10.0.4.4
 mpls te signal-protocol segment-routing
 mpls te tunnel-id 10
 mpls te path explicit-path PE1_PE4_Manual
 quit

ip vpn-instance vpna
 ipv4-family
  tnl-policy p1
 quit

tunnel-policy p1
 tunnel select-seq sr-te load-balance-number 1
 quit
```

### PE4

```text
explicit-path PE4_PE1_Manual
 next sid label 16003 type prefix
 next sid label 16005 type prefix
 next sid label 142337 type adjacency
 next sid label 142336 type adjacency
 next sid label 16001 type prefix
 quit

mpls
 mpls te
 quit

interface Tunnel10
 ip address unnumbered interface LoopBack0
 tunnel-protocol mpls te
 destination 10.0.1.1
 mpls te signal-protocol segment-routing
 mpls te tunnel-id 10
 mpls te path explicit-path PE4_PE1_Manual
 quit

ip vpn-instance vpna
 ipv4-family
  tnl-policy p1
 quit

tunnel-policy p1
 tunnel select-seq sr-te load-balance-number 1
 quit
```

## Etapa 3 - SR Policy Estática

### PE1

```text
segment-routing
 segment-list PE1_PE4_1
  index 10 sid label 16005
  index 20 sid label 142338
  index 30 sid label 16004
 quit

 segment-list PE1_PE4_2
  index 10 sid label 16005
  index 20 sid label 142337
  index 30 sid label 142338
 quit

 segment-list PE1_PE4_3
  index 10 sid label 16002
  index 20 sid label 16006
  index 30 sid label 142338
 quit

 sr-te policy p1 endpoint 10.0.4.4 color 100
  candidate-path preference 100
   segment-list PE1_PE4_1
   segment-list PE1_PE4_2
  quit
  candidate-path preference 50
   segment-list PE1_PE4_3
 quit
quit

tunnel-policy p1
 tunnel select-seq sr-te-policy load-balance-number 1 unmix
 quit

ip vpn-instance vpna
 tnl-policy p1
 ipv4-family
  default-color 100
 quit
```

### PE4

```text
segment-routing
 segment-list PE4_PE1_1
  index 10 sid label 16003
  index 20 sid label 16005
  index 30 sid label 142336
 quit

 segment-list PE4_PE1_2
  index 10 sid label 16006
  index 20 sid label 142337
  index 30 sid label 142336
 quit

 segment-list PE4_PE1_3
  index 10 sid label 16006
  index 20 sid label 142336
  index 30 sid label 16001
 quit

 sr-te policy p1 endpoint 10.0.1.1 color 100
  candidate-path preference 100
   segment-list PE4_PE1_1
   segment-list PE4_PE1_2
  quit
  candidate-path preference 50
   segment-list PE4_PE1_3
 quit
quit

tunnel-policy p1
 tunnel select-seq sr-te-policy load-balance-number 1 unmix
 quit

ip vpn-instance vpna
 tnl-policy p1
 ipv4-family
  default-color 100
 quit
```

## Etapa 4 - Proteção com SBFD e Hot Standby

### PE1

```text
bfd
 quit

sbfd
 reflector discriminator 10.0.1.1
 quit

segment-routing
 sr-te-policy seamless-bfd enable
 sr-te-policy backup hot-standby enable
 quit
```

### PE4

```text
bfd
 quit

sbfd
 reflector discriminator 10.0.4.4
 quit

segment-routing
 sr-te-policy seamless-bfd enable
 sr-te-policy backup hot-standby enable
 quit
```
