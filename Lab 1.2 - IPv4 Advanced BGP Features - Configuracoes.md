# Lab 1.2 - IPv4 Advanced BGP Features

Consolidação das linhas de configuração do laboratório, agrupadas por equipamento e separadas por função.

## Observações

- O conteúdo abaixo foi revisado a partir do `.docx` atualizado deste laboratório.
- Os blocos foram reorganizados com identação e espaçamento para facilitar leitura e aplicação.
- Comandos de verificação, testes de conectividade e saídas operacionais foram omitidos.
- O roteiro atualizado menciona a desativação de DCN, mas não apresenta explicitamente a linha de comando correspondente; por isso ela não foi consolidada abaixo.

## PE1

### Inicial

```text
system-view immediately
undo pnp enable

interface LoopBack0
 ip address 10.0.1.1 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.1 30
 quit

ip route-static 10.0.2.2 32 10.0.0.2
```

### BGP de Borda

```text
bgp 65001
 router-id 10.0.1.1
 peer 10.0.2.2 as-number 65100
 peer 10.0.2.2 connect-interface LoopBack0
 peer 10.0.2.2 password cipher Huawei@123
 peer 10.0.2.2 valid-ttl-hops 255
 quit
```

### Redes de Serviço e Comunidade

```text
interface LoopBack1
 ip address 10.1.1.1 32
 quit

interface LoopBack2
 ip address 10.2.1.1 32
 quit

bgp 65001
 network 10.1.1.1 32
 network 10.2.1.1 32
 peer 10.0.2.2 advertise-community
 quit

ip ip-prefix Com index 10 permit 10.1.1.1 32

route-policy Attr permit node 10
 if-match ip-prefix Com
 apply community 65001:1
 quit

route-policy Attr permit node 100
 quit

bgp 65001
 peer 10.0.2.2 route-policy Attr export
 quit
```

## PE2

### Inicial

```text
system-view immediately
undo pnp enable

interface LoopBack0
 ip address 10.0.2.2 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.2 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.9 30
 quit

ip route-static 10.0.1.1 32 10.0.0.1
```

### OSPF de Backbone

```text
ospf 1 router-id 10.0.2.2
 area 0.0.0.0
 quit
quit

interface LoopBack0
 ospf enable 1 area 0.0.0.0
 quit

interface GigabitEthernet3/0/1
 ospf enable 1 area 0.0.0.0
 quit
```

### BGP de Borda e iBGP

```text
bgp 65100
 router-id 10.0.2.2
 peer 10.0.1.1 as-number 65001
 peer 10.0.1.1 connect-interface LoopBack0
 peer 10.0.1.1 password cipher Huawei@123
 peer 10.0.1.1 valid-ttl-hops 255
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 peer 10.0.6.6 next-hop-local
 peer 10.0.6.6 advertise-community
 peer 10.0.1.1 advertise-community
 quit
```

### Política de Roteamento

```text
ip community-filter basic OA index 10 permit 65002:1
ip community-filter basic OA index 20 permit 65003:1
ip as-path-filter Finance permit 65002$

route-policy Finance permit node 10
 if-match community-filter OA
 quit

route-policy Finance deny node 20
 if-match as-path-filter Finance
 quit

route-policy Finance permit node 30
 quit

bgp 65100
 peer 10.0.1.1 route-policy Finance export
 quit
```

## PE3

### Inicial

```text
system-view immediately
undo pnp enable

interface LoopBack0
 ip address 10.0.3.3 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.25 30
 quit

ip route-static 10.0.4.4 32 10.0.0.26
```

### BGP de Borda

```text
bgp 65002
 router-id 10.0.3.3
 peer 10.0.4.4 as-number 65100
 peer 10.0.4.4 connect-interface LoopBack0
 peer 10.0.4.4 password cipher Huawei@123
 peer 10.0.4.4 valid-ttl-hops 255
 quit
```

### Redes de Serviço e Comunidade

```text
interface LoopBack1
 ip address 10.1.3.3 32
 quit

interface LoopBack2
 ip address 10.2.3.3 32
 quit

bgp 65002
 network 10.1.3.3 32
 network 10.2.3.3 32
 peer 10.0.4.4 advertise-community
 quit

ip ip-prefix Com index 10 permit 10.1.3.3 32

route-policy Attr permit node 10
 if-match ip-prefix Com
 apply community 65002:1
 quit

route-policy Attr permit node 100
 quit

bgp 65002
 peer 10.0.4.4 route-policy Attr export
 quit
```

### Rota Adicional para Teste de ORF

```text
interface LoopBack3
 ip address 10.3.3.3 32
 quit

bgp 65002
 network 10.3.3.3 32
 quit
```

## PE4

### Inicial

```text
system-view immediately
undo pnp enable

interface LoopBack0
 ip address 10.0.4.4 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.26 30
 quit

interface GigabitEthernet3/0/1
 ip address 10.0.0.22 255.255.255.252
 quit

ip route-static 10.0.3.3 32 10.0.0.25
```

### OSPF de Backbone

```text
ospf 1 router-id 10.0.4.4
 area 0.0.0.0
 quit
quit

interface LoopBack0
 ospf enable 1 area 0.0.0.0
 quit

interface GigabitEthernet3/0/1
 ospf enable 1 area 0.0.0.0
 quit
```

### BGP de Borda e iBGP

```text
bgp 65100
 router-id 10.0.4.4
 peer 10.0.3.3 as-number 65002
 peer 10.0.3.3 connect-interface LoopBack0
 peer 10.0.3.3 password cipher Huawei@123
 peer 10.0.3.3 valid-ttl-hops 255
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 peer 10.0.6.6 next-hop-local
 peer 10.0.6.6 advertise-community
 peer 10.0.3.3 advertise-community
 quit
```

### Política de Roteamento

```text
ip community-filter basic OA index 10 permit 65001:1
ip community-filter basic OA index 20 permit 65003:1
ip as-path-filter Finance permit 65001$

route-policy Finance permit node 10
 if-match community-filter OA
 quit

route-policy Finance deny node 20
 if-match as-path-filter Finance
 quit

route-policy Finance permit node 30
 quit

bgp 65100
 peer 10.0.3.3 route-policy Finance export
 quit
```

## P1

### Inicial

```text
system-view immediately
undo pnp enable

interface LoopBack0
 ip address 10.0.5.5 32
 quit

interface GigabitEthernet3/0/0
 ip address 10.0.0.13 30
 quit

ip route-static 10.0.6.6 32 10.0.0.14
```

### BGP de Borda

```text
bgp 65003
 router-id 10.0.5.5
 peer 10.0.6.6 as-number 65100
 peer 10.0.6.6 connect-interface LoopBack0
 peer 10.0.6.6 password cipher Huawei@123
 peer 10.0.6.6 valid-ttl-hops 255
 quit
```

### Redes de Serviço e Comunidade

```text
interface LoopBack1
 ip address 10.1.5.5 32
 quit

interface LoopBack2
 ip address 10.2.5.5 32
 quit

bgp 65003
 network 10.1.5.5 32
 network 10.2.5.5 32
 peer 10.0.6.6 advertise-community
 quit

ip ip-prefix Com index 10 permit 10.1.5.5 32

route-policy Attr permit node 10
 if-match ip-prefix Com
 apply community 65003:1
 quit

route-policy Attr permit node 100
 quit

bgp 65003
 peer 10.0.6.6 route-policy Attr export
 quit
```

### ORF

```text
ip ip-prefix Branch index 10 permit 10.1.1.1 32
ip ip-prefix Branch index 20 permit 10.2.1.1 32
ip ip-prefix Branch index 30 permit 10.1.3.3 32
ip ip-prefix Branch index 40 permit 10.2.3.3 32

bgp 65003
 peer 10.0.6.6 ip-prefix Branch import
 peer 10.0.6.6 capability-advertise orf ip-prefix send
 quit
```

## P2

### Inicial

```text
system-view immediately
undo pnp enable

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
 ip address 10.0.0.21 255.255.255.252
 quit

ip route-static 10.0.5.5 32 10.0.0.13
```

### OSPF de Backbone

```text
ospf 1 router-id 10.0.6.6
 area 0.0.0.0
 quit
quit

interface LoopBack0
 ospf enable 1 area 0.0.0.0
 quit

interface GigabitEthernet3/0/1
 ospf enable 1 area 0.0.0.0
 quit

interface GigabitEthernet3/0/2
 ospf enable 1 area 0.0.0.0
 quit
```

### BGP de Borda e RR

```text
bgp 65100
 router-id 10.0.6.6
 peer 10.0.5.5 as-number 65003
 peer 10.0.5.5 connect-interface LoopBack0
 peer 10.0.5.5 password cipher Huawei@123
 peer 10.0.5.5 valid-ttl-hops 255
 group BB internal
 peer BB connect-interface LoopBack0
 peer BB next-hop-local
 peer 10.0.2.2 group BB
 peer 10.0.4.4 group BB
 ipv4-family unicast
  peer BB reflect-client
 quit
 peer 10.0.5.5 advertise-community
 peer BB advertise-community
 peer 10.0.5.5 capability-advertise orf ip-prefix receive
 quit
```

## Resumo de Endereços Loopback

| Nó | LoopBack0 | LoopBack1 | LoopBack2 | LoopBack3 |
|---|---|---|---|---|
| PE1 | 10.0.1.1/32 | 10.1.1.1/32 | 10.2.1.1/32 | — |
| PE2 | 10.0.2.2/32 | — | — | — |
| PE3 | 10.0.3.3/32 | 10.1.3.3/32 | 10.2.3.3/32 | 10.3.3.3/32 |
| PE4 | 10.0.4.4/32 | — | — | — |
| P1 | 10.0.5.5/32 | 10.1.5.5/32 | 10.2.5.5/32 | — |
| P2 | 10.0.6.6/32 | — | — | — |

- LoopBack0: identificação do roteador, no formato 10.0.X.X/32.
- LoopBack1: rede de serviço OA, no formato 10.1.X.X/32.
- LoopBack2: rede de serviço Financeiro, no formato 10.2.X.X/32.
- LoopBack3: rota adicional usada no teste de ORF, presente apenas no PE3.
