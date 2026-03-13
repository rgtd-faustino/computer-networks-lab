# Computer Networks Lab — ISEL

> Infraestrutura de rede corporativa completa desenvolvida em 4 fases progressivas com Cisco Packet Tracer.

**Unidade Curricular:** Redes de Computadores · ISEL 2024/2025  
**Equipa:** João Póvoa (51392) · Rafael Faustino (51394) · Sofia Salgado (51694)

---

## Visão Geral

Este projeto constrói uma rede corporativa funcional do zero, fase a fase. Cada fase estende a anterior — a topologia final integra tudo: LANs com clientes, routing inter-redes, serviços centralizados e conectividade com o exterior.

<p align="center">
  <img src="images/network-topology.png" alt="Topologia da rede no Cisco Packet Tracer" width="800"/>
</p>

---

## Fases do Projeto

### Fase 1 — Web Server & TCP/IP Client
Instalação e configuração de um servidor web, com desenvolvimento de um cliente HTTP personalizado a comunicar via TCP/IP.

---

### Fase 2 — Connecting Devices (LAN A & B)

Primeira configuração no Cisco Packet Tracer: dois segmentos de rede ligados por um router.

**Subnetting aplicado** a partir de `10.0.6.0/24`:

| Sub-rede | Intervalo | Máscara | Uso |
|---|---|---|---|
| `10.0.6.0/25` | .0 – .127 | 255.255.255.128 | LAN B |
| `10.0.6.128/25` | .128 – .255 | 255.255.255.128 | LAN A |

**Router R1:**
- `FastEthernet0/0` → `10.0.6.126` (gateway LAN B)  
- `FastEthernet1/0` → `10.0.6.254` (gateway LAN A)

**Validação:** ping 100% entre dispositivos das duas LANs. Tabela de routing confirmada com `show ip route`.

---

### Fase 3 — Connecting Multiple Networks

Expansão para topologia multi-router com LAN de servidores e acesso externo simulado.

**Subnetting hierárquico (VLSM)** a partir de `10.0.6.0/24`:

| Sub-rede | Notação CIDR | Hosts úteis | Uso |
|---|---|---|---|
| `10.0.6.0` | `/25` | 126 | LAN A (80 clientes) |
| `10.0.6.128` | `/26` | 62 | LAN B (40 clientes) |
| `10.0.6.224` | `/27` | 30 | LAN Server |
| `10.0.6.192` | `/30` | 2 | Transit A (R1 ↔ R2) |

> O número de clientes foi calculado a partir dos números de aluno:  
> `(51392 + 51394 + 51694) mod 100 = 80` → LAN A: 80, LAN B: 40

**Routing estático configurado:**

```
# Router R1
ip route 10.0.6.224 255.255.255.224 10.0.6.194   # → LAN Server via R2
ip route 8.8.8.0 255.255.255.252 10.0.6.194       # → default (Internet)

# Router R2
ip route 10.0.6.0 255.255.255.128 10.0.6.193      # → LAN A via R1
ip route 10.0.6.128 255.255.255.192 10.0.6.193    # → LAN B via R1
```

**Validação:** ping de Laptop1 (LAN A) a `8.8.8.1` (externo) com sucesso. Conectividade total entre todas as sub-redes verificada.

---

### Fase 4 — Deploy Services

Implementação de serviços de rede sobre a topologia da Fase 3. Os clientes deixam de ter IPs estáticos — tudo passa a ser atribuído automaticamente.

#### DHCP

Servidor em `10.0.6.225` com três pools:

| Pool | Start IP | Gateway | DNS |
|---|---|---|---|
| `LAN_A` | `10.0.6.1` | `10.0.6.126` | `10.0.6.226` |
| `LAN_B` | `10.0.6.129` | `10.0.6.190` | `10.0.6.226` |
| `serverPool` | `10.0.6.224` | `200.0.3.1` | `200.0.3.101` |

Como o servidor DHCP está numa sub-rede separada dos clientes, foi necessário configurar **DHCP Relay Agent** em cada interface do router voltada para as LANs de clientes:

```
ip helper-address 10.0.6.225
```

O router converte os broadcasts DHCP em unicast direcionados ao servidor, incluindo informação sobre a subnet de origem para que o pool correto seja selecionado.

#### DNS

Servidor em `10.0.6.226`. Registo configurado:

```
www.company.com  →  A Record  →  10.0.6.227
```

#### HTTP / HTTPS

Servidor web em `10.0.6.227` com as portas 80 e 443 ativas.

**Validação:**
```
nslookup www.company.com   # resolve para 10.0.6.227
ping www.company.com        # 4/4 pacotes, 0% perda
http://www.company.com      # página carregada com sucesso em PC1 e PC0
```

#### Tabela ARP — experiência observada

- Estado inicial: `No ARP Entries Found` (sem comunicação prévia)
- Após ping do PC1 ao servidor DHCP: entrada `10.0.6.254` (gateway do router) criada dinamicamente
- Após desligar e religar a interface: tabela limpa novamente

---

## Stack / Ferramentas

| Ferramenta | Uso |
|---|---|
| Cisco Packet Tracer 8.2 | Simulação completa da rede |
| Wireshark | Análise de protocolos (Fase 1) |
| CLI Cisco IOS | Configuração de routers e verificação |

**Protocolos abordados:** IPv4, ICMP, ARP, DHCP (DORA), DNS, HTTP, HTTPS, routing estático

---

## Referências

- Kurose, J. F.; Ross, K. W. (2021). *Computer Networking: A Top-Down Approach* (8ª ed.). Pearson.
- Cisco Systems. *Cisco Packet Tracer*, versão 8.2.
- Slides da UC: Chapter 4 – Network Layer; Chapter 5 – Link Layer. ISEL 2025.
