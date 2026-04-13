# Lab 1.1 - IPv4 Routing

Consolidação das linhas de configuração do laboratório, agrupadas por roteador.

## Observações

- O conteúdo abaixo consolida a configuração final desejada do laboratório.
- Os comandos temporários usados apenas para demonstrar a convergência via BFD foram separados no fim.
- Os prompts (`<PE1>`, `[PE1]`) foram removidos para facilitar a aplicação direta.

## PE1

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.1.1 32
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.1 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.5 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface LoopBack2
 ip address 10.2.16.16 32

ospf enable 1 area 0
quit

interface LoopBack0

ospf enable 1 area 0
quit

ospf 1 router-id 10.0.1.1
 area 0
quit

bfd
quit

ospf 1
 bfd all-interfaces enable
quit
```

## PE2

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.2.2 32

ospf enable 1 area 0
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.2 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.9 30

ospf enable 1 area 0

ospf cost 2

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

ospf 1 router-id 10.0.2.2
 area 0
quit

ospf 1
 frr
 loop-free-alternate
quit
 bfd all-interfaces enable
quit

bfd
quit
```

## PE3

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.3.3 32

ospf enable 1 area 1
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.25 30

isis enable 1
quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.18 30

ospf enable 1 area 1

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface LoopBack3
 ip address 10.3.3.3 255.255.255.0

ospf network-type broadcast

ospf enable 1 area 0.0.0.1
quit

ospf 1 router-id 10.0.3.3
 area 1
quit

isis 1
 is-level level-1-2
 network-entity 49.0001.0000.0000.0003.00
 default-route-advertise always level-1-2
quit

bfd
quit

ospf 1
 bfd all-interfaces enable
quit
```

## PE4

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.4.4 32

isis enable 1
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.26 30

isis enable 1
quit

isis 1
 is-level level-1
 network-entity 49.0001.0000.0000.0004.00
quit
```

## P1

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.5.5 32

ospf enable 1 area 0
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.13 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.6 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface GigabitEthernet3/0/2
 ip address 10.0.0.17 30

ospf enable 1 area 1

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

ospf 1 router-id 10.0.5.5
 area 0
quit
 area 1
quit
 maximum load-balancing 1
quit

ip ip-prefix 1 index 10 deny 10.3.3.0 24 greater-equal 24 less-equal 24

ip ip-prefix 1 index 20 permit 0.0.0.0 0 less-equal 32

ospf 1
 area 1
 filter ip-prefix 1 export
quit
quit

bfd
quit

ospf 1
 bfd all-interfaces enable
quit
```

## P2

```text
system-view immediately

undo pnp enable

interface LoopBack0
 ip address 10.0.6.6 32

ospf enable 1 area 0
quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.14 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.10 30

ospf enable 1 area 0

ospf bfd enable

ospf bfd min-tx-interval 500 min-rx-interval 500 detect-multiplier 3
quit

interface LoopBack2
 ip address 10.2.16.16 32

ospf enable 1 area 0
quit

ospf 1 router-id 10.0.6.6
 area 0
quit

bfd
quit

ospf 1
 bfd all-interfaces enable
quit
```

## Comandos TemporÃ¡rios de ValidaÃ§Ã£o do BFD no PE2

Esses comandos aparecem no roteiro apenas para simular falha e observar a convergï¿½ncia. Eles nï¿½o fazem parte da configuração final permanente.

```text
acl number 3000
 rule 1 permit udp destination-port eq 3784
quit

traffic classifier bfd operator or
 if-match acl 3000
quit

traffic behavior bfd
 deny
quit

traffic policy bfd
 classifier bfd behavior bfd
quit

interface GigabitEthernet3/0/0
 traffic-policy bfd inbound
quit

acl 3000
 rule 2 permit ospf
quit

interface GigabitEthernet3/0/0
 undo traffic-policy bfd inbound
quit
```

