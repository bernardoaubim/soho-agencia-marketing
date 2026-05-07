# Política de Acesso entre VLANs – ACL Rules

---

## Visão Geral

O controle de tráfego inter-VLAN é implementado por meio de **ACLs Estendidas** aplicadas nas SVIs do `SW-CORE-L3`. A lógica segue o princípio de **menor privilégio**: cada VLAN recebe apenas o acesso estritamente necessário para sua função.

---

## Matriz de Acesso

| Origem \ Destino      | V10 Oper. | V20 Dir. | V30 Serv. | V31 Impr. | V40 IoT | V50 Guest | V99 MGMT | Internet |
|-----------------------|:---------:|:--------:|:---------:|:---------:|:-------:|:---------:|:--------:|:--------:|
| **V10 – Operacional** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V20 – Diretoria**   | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V30 – Servidores**  | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V31 – Impressoras** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **V40 – IoT**         | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |
| **V50 – Guest**       | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| **V99 – MGMT/TI**     | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

> ✅ = Permitido | ❌ = Bloqueado pela ACL

---

## ACLs por VLAN

### ACL-VLAN40-IoT — Aplicada na SVI VLAN 40 (entrada)

**Objetivo:** Isolar dispositivos IoT. Permitir apenas comunicação intra-VLAN e acesso ao segmento de gerenciamento (VLAN 99) para manutenção remota. Bloquear qualquer outro destino.

```
 permit ip 192.168.40.0 0.0.0.255 10.0.99.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 10.0.0.0 0.0.255.255
 deny ip 192.168.40.0 0.0.0.255 172.16.0.0 0.0.255.255
 deny ip 192.168.40.0 0.0.0.255 192.168.0.0 0.0.255.255
 deny ip any any
```

**Aplicação:**
```
interface vlan 40
 ip access-group ACL-VLAN40-IoT in
```

**Justificativa:** Câmeras IP e sensores não necessitam de acesso à internet nem a outras VLANs corporativas. Liberar internet para IoT criaria um vetor de ataque (botnet). O acesso à VLAN 99 é necessário para que a equipe de TI gerencie os dispositivos remotamente.

---

### ACL-VLAN50-Guest — Aplicada na SVI VLAN 50 (entrada)

**Objetivo:** Restringir visitantes à internet. Nenhum acesso a redes internas.

```
ip access-list extended ACL-VLAN50-Guest
 deny ip 172.16.50.0 0.0.0.255 192.168.0.0 0.0.255.255
 deny ip 172.16.50.0 0.0.0.255 172.16.0.0 0.0.255.255
 deny ip 172.16.50.0 0.0.0.255 10.0.0.0 0.0.255.255
 permit ip 172.16.50.0 0.0.0.255 any
```

**Aplicação:**
```
interface vlan 50
 ip access-group ACL-VLAN50-Guest in
```

**Justificativa:** A rede de visitantes deve oferecer conectividade à internet sem qualquer visibilidade das redes internas. O bloqueio explícito de todos os blocos RFC 1918 garante que, mesmo que um visitante tente alcançar um IP interno, o tráfego seja descartado antes de ser roteado.

---

### VLANs 10, 20, 30, 31 — Sem restrições de saída

As VLANs corporativas (Operacional, Diretoria, Servidores e Impressoras) têm acesso irrestrito entre si e à internet. O controle de quem acessa *essas* VLANs é feito pelas ACLs aplicadas nas VLANs de origem (40 e 50).

---

### VLAN 99 – MGMT/TI — Acesso administrativo controlado

A VLAN 99 permite acesso a todas as VLANs corporativas e à internet, mas **não acessa VLAN 50 (Guest)**. 

```
ip access-list extended ACL-MGMT-ADMIN
 permit ip 10.0.99.0 0.0.0.255 any
```

**Aplicação:**
```
interface vlan 99
 ip access-group ACL-VLAN99-MGMT in
```

---

## Segurança Adicional nos Switches de Acesso

### Port Security (SW-ACESSO-A e SW-ACESSO-B)

Configurado em todas as portas de acesso ativas para prevenir conexão de dispositivos não autorizados.

```
! Exemplo aplicado nas portas de acesso
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
```

| Parâmetro         | Valor    | Descrição                                        |
|-------------------|----------|--------------------------------------------------|
| Maximum MACs      | 1        | Apenas 1 dispositivo por porta                   |
| Modo sticky       | Ativado  | MAC aprendido automaticamente e salvo na config  |
| Violação          | Shutdown | Porta desligada automaticamente em caso de fraude|

---

### BPDU Guard + PortFast

Aplicado em todas as portas de acesso para proteger contra ataques de STP e acelerar a convergência.

```
interface range fa0/1-24
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

### Portas Não Utilizadas

Todas as portas livres (sem dispositivo conectado) estão administrativamente desativadas e com descrição padronizada.

```
interface range fa0/11-18
 description PORTA-LIVRE-DESATIVADA
 shutdown
```

---

## Hardening Administrativo

| Medida                     | Configuração                                   |
|----------------------------|------------------------------------------------|
| SSH v2 habilitado          | `transport input ssh` em todas as VTYs         |
| Telnet bloqueado           | `transport input none` (implícito após SSH)    |
| Login local com senha      | `username NetAdmin secret <senha>`             |
| Proteção brute-force       | `login block-for 60 attempts 3 within 30` (roteador) |
| Banner MOTD                | Aviso de acesso não autorizado em todos os equipamentos |
| Senha enable criptografada | `enable secret <senha>`                        |
| `service password-encryption` | Ativa em todos os dispositivos            |

---
