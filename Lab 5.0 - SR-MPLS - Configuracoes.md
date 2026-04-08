# Lab 5.0 - SR-MPLS

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
interface GigabitEthernet0/5/0
ip address 10.0.0.1 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.5 30
quit
isis 1
is-level level-2
cost-style wide
network-entity 49.0001.0001.0001.0001.00
is-name PE1
interface LoopBack0
isis enable 1
quit
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 1
quit
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:10
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.1.1 32
bgp 65100
router-id 10.0.1.1
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
system-view immediately
undo dcn
interface LoopBack0
ip address 10.0.1.1 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.1 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.5 30
quit
isis 1
is-level level-2
cost-style wide
network-entity 49.0001.0001.0001.0001.00
is-name PE1
interface LoopBack0
isis enable 1
quit
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 1
quit
explicit-path PE1_PE4_Manual
next sid label 16005 type prefix
next sid label 142337 type adjacency
next sid label 142338 type adjacency
mpls
mpls te
interface Tunnel10
ip address unnumbered interface LoopBack0
tunnel-protocol mpls te
destination 10.0.4.4
mpls te signal-protocol segment-routing
mpls te tunnel-id 10
mpls te path explicit-path PE1_PE4_Manual
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:10
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.1.1 32
bgp 65100
router-id 10.0.1.1
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
tunnel-policy p1
tunnel select-seq sr-te load-balance-number 1
quit
ip vpn-instance vpna
ipv4-family
tnl-policy p1
system-view immediately
undo dcn
interface LoopBack0
ip address 10.0.1.1 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.1 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.5 30
quit
isis 1
is-level level-2
cost-style wide
network-entity 49.0001.0001.0001.0001.00
is-name PE1
interface LoopBack0
isis enable 1
quit
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 1
quit
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
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:10
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.1.1 32
bgp 65100
router-id 10.0.1.1
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
tunnel-policy p1
tunnel select-seq sr-te-policy load-balance-number 1 unmix
ip vpn-instance vpna
tnl-policy p1
ip vpn-instance vpna
ipv4-family
default-color 100
bfd
quit
sbfd
reflector discriminator 10.0.1.1
quit
segment-routing
sr-te-policy seamless-bfd enable
quit
segment-routing
sr-te-policy backup hot-standby enable
```

## PE2

```text
system-view immediately
interface LoopBack0
ip address 10.0.2.2 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.2 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
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
system-view immediately
interface LoopBack0
ip address 10.0.2.2 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.2 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
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
system-view immediately
interface LoopBack0
ip address 10.0.2.2 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.2 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
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
```

## PE3

```text
system-view immediately
interface LoopBack0
ip address 10.0.3.3 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.25 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 3
quit
router-id 10.0.3.3
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
system-view immediately
interface LoopBack0
ip address 10.0.3.3 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.25 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 3
quit
router-id 10.0.3.3
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
system-view immediately
interface LoopBack0
ip address 10.0.3.3 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.25 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 3
quit
router-id 10.0.3.3
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
```

## PE4

```text
system-view immediately
interface LoopBack0
ip address 10.0.4.4 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.26 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 4
quit
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:40
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.4.4 32
bgp 65100
router-id 10.0.4.4
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
system-view immediately
interface LoopBack0
ip address 10.0.4.4 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.26 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 4
quit
explicit-path PE4_PE1_Manual
next sid label 16003 type prefix
next sid label 16005 type prefix
next sid label 142337 type adjacency
next sid label 142336 type adjacency
next sid label 16001 type prefix
mpls
mpls te
interface Tunnel10
ip address unnumbered interface LoopBack0
tunnel-protocol mpls te
destination 10.0.1.1
mpls te signal-protocol segment-routing
mpls te tunnel-id 10
mpls te path explicit-path PE4_PE1_Manual
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:40
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.4.4 32
bgp 65100
router-id 10.0.4.4
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
tunnel-policy p1
tunnel select-seq sr-te load-balance-number 1
quit
ip vpn-instance vpna
ipv4-family
tnl-policy p1
system-view immediately
interface LoopBack0
ip address 10.0.4.4 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.26 30
quit
interface GigabitEthernet0/5/1
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
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
interface LoopBack 0
isis prefix-sid index 4
quit
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
ip vpn-instance vpna
ipv4-family
route-distinguisher 100:40
vpn-target 100:1020 both
quit
interface LoopBack 1
ip binding vpn-instance vpna
ip address 10.1.4.4 32
bgp 65100
router-id 10.0.4.4
peer 10.0.5.5 as-number 65100
peer 10.0.5.5 connect-interface LoopBack0
peer 10.0.6.6 as-number 65100
peer 10.0.6.6 connect-interface LoopBack0
ipv4-family vpnv4
peer 10.0.5.5 enable
peer 10.0.6.6 enable
bgp 65100
ipv4-family vpn-instance vpna
import-route direct
tunnel-policy p1
tunnel select-seq sr-te-policy load-balance-number 1 unmix
ip vpn-instance vpna
tnl-policy p1
ip vpn-instance vpna
ipv4-family
default-color 100
bfd
quit
sbfd
reflector discriminator 10.0.4.4
quit
segment-routing
sr-te-policy seamless-bfd enable
quit
segment-routing
sr-te-policy backup hot-standby enable
```

## P1

```text
system-view immediately
interface LoopBack0
ip address 10.0.5.5 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.13 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.6 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
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
system-view immediately
interface LoopBack0
ip address 10.0.5.5 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.13 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.6 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
isis prefix-sid index 5
quit
segment-routing
ipv4 adjacency local-ip-addr 10.0.0.6 remote-ip-addr 10.0.0.5 sid 142336
ipv4 adjacency local-ip-addr 10.0.0.13 remote-ip-addr 10.0.0.14 sid 142337
ipv4 adjacency local-ip-addr 10.0.0.17 remote-ip-addr 10.0.0.18 sid 142338
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
system-view immediately
interface LoopBack0
ip address 10.0.5.5 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.13 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.6 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
isis prefix-sid index 5
quit
segment-routing
ipv4 adjacency local-ip-addr 10.0.0.6 remote-ip-addr 10.0.0.5 sid 142336
ipv4 adjacency local-ip-addr 10.0.0.13 remote-ip-addr 10.0.0.14 sid 142337
ipv4 adjacency local-ip-addr 10.0.0.17 remote-ip-addr 10.0.0.18 sid 142338
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
interface GigabitEthernet0/5/2
quit
interface GigabitEthernet0/5/0
```

## P2

```text
system-view immediately
interface LoopBack0
ip address 10.0.6.6 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.14 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.10 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
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
system-view immediately
interface LoopBack0
ip address 10.0.6.6 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.14 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.10 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
isis prefix-sid index 6
quit
segment-routing
ipv4 adjacency local-ip-addr 10.0.0.10 remote-ip-addr 10.0.0.9 sid 142336
ipv4 adjacency local-ip-addr 10.0.0.14 remote-ip-addr 10.0.0.13 sid 142337
ipv4 adjacency local-ip-addr 10.0.0.21 remote-ip-addr 10.0.0.22 sid 142338
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
system-view immediately
interface LoopBack0
ip address 10.0.6.6 32
quit
interface GigabitEthernet0/5/0
ip address 10.0.0.14 30
quit
interface GigabitEthernet0/5/1
ip address 10.0.0.10 30
quit
interface GigabitEthernet0/5/2
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
interface GigabitEthernet0/5/0
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/1
isis enable 1
isis circuit-type p2p
quit
interface GigabitEthernet0/5/2
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
interface LoopBack 0
isis prefix-sid index 6
quit
segment-routing
ipv4 adjacency local-ip-addr 10.0.0.10 remote-ip-addr 10.0.0.9 sid 142336
ipv4 adjacency local-ip-addr 10.0.0.14 remote-ip-addr 10.0.0.13 sid 142337
ipv4 adjacency local-ip-addr 10.0.0.21 remote-ip-addr 10.0.0.22 sid 142338
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
```


