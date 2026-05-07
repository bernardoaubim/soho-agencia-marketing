# Projeto SOHO – Agência de Marketing Digital

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Autor](https://img.shields.io/badge/Autor-bernardoaubim-blue?style=for-the-badge&logo=github)

---

## Sobre o Projeto

Este projeto simula a infraestrutura de rede completa de uma agência de marketing digital com 20 funcionários, servidores internos e dispositivos IoT. O objetivo é demonstrar competência técnica em segmentação de redes, segurança por camadas, roteamento inter-VLAN e boas práticas de administração Cisco.

O cenário foi desenhado com foco em ambientes reais: cada decisão técnica tem uma justificativa de negócio, e a rede foi construída para ser escalável, segura e gerenciável remotamente.

O foco principal é:

- Segmentação de rede com VLANs;
- Controle de acesso entre departamentos;
- Hardening básico de switches;
- Segurança de portas com Port Security;
- Implementação de ACLs;
- Gerenciamento centralizado via switch Layer 3.

---

## Estrutura do Repositório

```
 soho-agencia-marketing/
├──   configs/
│   ├── rtr-agencia-borda.txt       # show running-config do Roteador
│   ├── sw-core-l3.txt              # show running-config do Core
│   ├── sw-acesso-a.txt             # show running-config do Switch A
│   └── sw-acesso-b.txt             # show running-config do Switch B
├──   diagrama/
│   └── topologia.png               # Topologia no Packet Tracer
├──  docs/
│   ├── tabela-endereçamento.md     # Tabela de endereçamento completa
│   ├── acl-regras.md               # Política de acesso entre VLANs
│   └── teste-resultados.md          # Resultados dos testes de conectividade
├──  packet-tracer/
│   └── soho-agencia.pkt            # Arquivo do Packet Tracer
└── README.md
```

---

##  Topologia da Rede

```
                         Internet
                            │
                  [RTR-AGENCIA-BORDA]          ← ISR 4321
                  WAN: 200.10.10.2/30
                  LAN: 10.0.0.1/30
                            │
                      [SW-CORE-L3]             ← Catalyst 3650 (Layer 3)
                      10.0.0.2/30
                     /              \
              G1/0/2                G1/0/3
         VLANs 10,30,31,99      VLANs 20,40,50
                /                        \
        [SW-ACESSO-A]              [SW-ACESSO-B]   ← Catalyst 2960
               │                          │
    ┌──────────┼──────┐         ┌─────────┼──────┐
  VLAN 10   VLAN 30  VLAN 99  VLAN 20  VLAN 40  VLAN 50
 Operacion. Servidor   TI    Diretoria   IoT    [AP WiFi]
  10 PCs   Web+NAS  3 PCs TI  5 Laptops Câmeras Visitantes
```

---

## Plano de Endereçamento – VLANs

| VLAN |     Nome    |       Rede      |    Gateway   |                 Dispositivos               |
|------|-------------|-----------------|--------------|--------------------------------------------|
|  10  | OPERACIONAL | 192.168.10.0/24 | 192.168.10.1 | 10 PCs – SW-A Fa0/1-10                     |
|  20  | DIRETORIA   | 192.168.20.0/24 | 192.168.20.1 | 5 Laptops – SW-B Fa0/1-5                   |
|  30  | SERVIDORES  | 192.168.30.0/24 | 192.168.30.1 | Web Server + NAS – SW-A Fa0/19-20          |
|  31  | IMPRESSORAS | 192.168.31.0/24 | 192.168.31.1 | 1 Impressora – SW-A Fa0/21                 |
|  40  | IoT         | 192.168.40.0/24 | 192.168.40.1 | Câmeras/Sensores – SW-B Fa0/18-24          |
|  50  | GUEST       | 172.16.50.0/24  | 172.16.50.1  | Visitantes via Wi-Fi – SW-B G0/2           |
|  99  | MGMT/TI     | 10.0.99.0/24    | 10.0.99.1    | PCs de TI + Gerenciamento – SW-A Fa0/22-24 |

---

## Política de Segurança – Matriz de Acesso

As seguintes políticas foram implementadas:

- VLAN de visitantes isolada da rede interna;
- Dispositivos IoT restritos;
- Controle de acesso entre setores utilizando ACLs;
- Gerenciamento separado na VLAN 99;
- Port Security nos switches de acesso;
- Shutdown automático em violação de segurança;
- Senhas configuradas para acesso administrativo;
- SSH habilitado para gerenciamento remoto seguro.

| Origem                | V10 Oper. | V20 Dir. | V30 Serv. | V31 Impr. | V40 IoT | V50 Guest | Internet |
|-----------------------|-----------|----------|-----------|-----------|---------|-----------|----------|
| **V10 – Operacional** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V20 – Diretoria**   | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V30 – Servidores**  | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V31 – Impressoras** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V40 – IoT**         | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| **V50 – Guest**       | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **V99 – MGMT/TI**     | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

> **IoT (VLAN 40)** acessa apenas a própria VLAN e a VLAN 99 para fins de gerenciamento.  
> **Guest (VLAN 50)** acessa exclusivamente a internet — sem acesso a nenhuma rede interna.

---

## Tecnologias e Conceitos Implementados

### Rede
- **Inter-VLAN Routing via Switch Layer 3** — roteamento no Catalyst 3650, sem sobrecarregar o roteador
- **DHCP com relay (ip helper-address)** — servidor DHCP centralizado no roteador, relay nas SVIs do Core
- **NAT PAT (overload)** — tradução de endereços para acesso à internet
- **Link ponto-a-ponto /30** — interconexão eficiente entre roteador e Core
- **Rotas estáticas** — roteamento entre redes internas e gateway de internet
- **Trunk com VLANs restritas** — cada trunk carrega apenas as VLANs necessárias

### Segurança
- **ACLs Estendidas** — políticas de controle de acesso inter-VLAN aplicadas nas SVIs
- **Port-Security com Sticky MAC** — bloqueio de dispositivos não autorizados nas portas
- **BPDU Guard + PortFast** — proteção contra ataques de STP nas portas de acesso
- **Rapid PVST+** — protocolo de spanning tree em todos os dispositivos
- **SSH v2 com login local** — acesso remoto seguro em todos os equipamentos
- **login block-for** — proteção automática contra brute-force no roteador
- **Portas não utilizadas desativadas** — todas as portas livres com shutdown e description

### Gerenciamento
- **VLAN 99 dedicada** — tráfego administrativo isolado em range 10.0.99.x
- **SVI de gerenciamento** — cada switch acessível via IP próprio (10.0.99.10 e 10.0.99.11)
- **Banner MOTD** — aviso de acesso não autorizado em todos os dispositivos
- **Descrições em todas as interfaces** — facilita identificação e troubleshooting

---

## Testes de Conectividade

| # | Teste | Esperado | Resultado |
|---|-------|----------|-----------|
| 1 | VLAN 10 → VLAN 30 (Servidor) | ✅ Funciona | ✅ |
| 2 | VLAN 10 → VLAN 20 (Diretoria) | ✅ Funciona | ✅ |
| 3 | VLAN 10 → Internet | ✅ Funciona via NAT | ✅ |
| 4 | VLAN 20 → VLAN 30 (Servidor) | ✅ Funciona | ✅ |
| 5 | VLAN 40 (IoT) → VLAN 10 | ❌ Bloqueado pela ACL | ❌ |
| 6 | VLAN 40 (IoT) → Internet | ❌ Bloqueado pela ACL | ❌ |
| 7 | VLAN 40 (IoT) → VLAN 99 | ✅ Funciona (gerenciamento) | ✅ |
| 8 | VLAN 50 (Guest) → Internet | ✅ Funciona via NAT | ✅ |
| 9 | VLAN 50 (Guest) → VLAN 10 | ❌ Bloqueado pela ACL | ❌ |
| 10 | SSH | ✅ Funciona | ✅ |
| 11 | Telnet  | ❌ Bloqueado | ❌ |

>  Prints de todos os testes disponíveis em `/docs/test-results.md`

---

##  Como Reproduzir o Lab

### Pré-requisitos
- [Cisco Packet Tracer](https://www.netacad.com/resources/lab-downloads) versão 8.x ou superior (gratuito com cadastro na NetAcad)

### Passo a passo
```bash
1. Clone este repositório
   git clone https://github.com/bernardoaubim/soho-agencia-marketing

2. Abra o Packet Tracer

3. Arquivo → Abrir → selecione packet-tracer/soho-agencia.pkt

4. Aguarde a inicialização dos dispositivos (30 segundos)

5. Teste os pings conforme a tabela acima
```

### Verificações recomendadas
```
! No SW-CORE-L3:
show vlan brief
show interfaces trunk
show ip route
show access-lists
show running-config | include access-group

! No RTR-AGENCIA-BORDA:
show ip nat translations
show ip dhcp binding
show access-lists
```

---

##  Equipamentos Utilizados

| Equipamento | Modelo | Função |
|-------------|--------|--------|
| Roteador | ISR 4321 | NAT, DHCP, Gateway WAN |
| Switch Core | Catalyst 3650 | Inter-VLAN L3 |
| Switch Acesso A | Catalyst 2960 | VLANs 10, 30, 31, 99 |
| Switch Acesso B | Catalyst 2960 | VLANs 20, 40, 50 |
| Access Point | AP genérico PT | Wi-Fi exclusivo VLAN 50 (visitantes) |

---

## Decisões de Projeto

Cada escolha técnica foi tomada com justificativa — não apenas para funcionar, mas para fazer sentido em um ambiente real.

**Por que Catalyst 3650 no Core em vez de 2960?**  
O 2960 não suporta roteamento Layer 3 (ip routing). Usar um 2960 como Core forçaria o uso de Router-on-a-Stick no roteador, criando um gargalo de tráfego. O 3650 faz o inter-VLAN localmente, liberando o roteador apenas para NAT e gateway WAN.

**Por que VLAN 31 separada para impressoras?**  
Impressoras e servidores web têm perfis de segurança completamente diferentes. Uma impressora comprometida não deve ter acesso ao servidor web. A separação permite aplicar políticas de ACL distintas e limitar o raio de explosão em caso de incidente.

**Por que IoT sem acesso à internet?**  
Câmeras IP e sensores não precisam de internet para funcionar — eles enviam dados para servidores internos. Liberar internet para IoT cria um vetor de ataque clássico (botnet Mirai usou exatamente essa vulnerabilidade). A exceção foi o acesso à VLAN 99 para permitir gerenciamento remoto dos dispositivos.

**Por que Wi-Fi exclusivo para visitantes (VLAN 50)?**  
A Diretoria opera via cabo para garantir estabilidade e segurança. Wi-Fi para usuários corporativos exigiria autenticação 802.1X para ser seguro — fora do escopo deste projeto. Limitar o AP à VLAN de visitantes simplifica a arquitetura e elimina riscos de VLAN hopping via wireless.

**Por que link /30 entre roteador e Core?**  
Uma rede /30 tem apenas 2 IPs utilizáveis — exatamente o necessário para um link ponto-a-ponto. Usar /24 nesse link desperdiçaria 252 IPs e aumentaria a superfície de ataque sem nenhum benefício.


---



##  Referências

- [Cisco Networking Academy – CCNA: Introduction to Networks](https://www.netacad.com)


---

## Próximos Passos

Em um ambiente de produção real, os seguintes itens seriam adicionados:

- Implementação de DHCP Snooping;
- Dynamic ARP Inspection;
- AAA Authentication;
- Syslog centralizado;
- SNMP seguro;
- Firewall dedicado;
- IDS/IPS.
- IPv6

---

## Autor

**Bernardo Aubim**  
Estudante de Segurança da Informação  
[![GitHub](https://img.shields.io/badge/GitHub-bernardoaubim-181717?style=flat&logo=github)](https://github.com/bernardoaubim)

---
