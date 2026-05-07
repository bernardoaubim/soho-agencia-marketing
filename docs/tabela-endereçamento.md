# Tabela de Endereçamento – SOHO Agência de Marketing Digital

---

## Infraestrutura de Rede – Links e SVIs

| Dispositivo       | Interface  | Endereço IP     | Máscara         | Descrição                          |
|-------------------|------------|-----------------|-----------------|------------------------------------|
| RTR-AGENCIA-BORDA | G0/0/1     | 200.10.10.2     |30 255.255.255.252  | Internet                      |
| RTR-AGENCIA-BORDA | G0/0/0    | 10.0.0.1        | /30 255.255.255.252 | Link ponto-a-ponto → SW-CORE-L3 |             |
| SW-CORE-L3        | G1/0/1     | 10.0.0.2        | /30 255.255.255.252 | Link ponto-a-ponto → RTR-BORDA  |
| SW-CORE-L3        | G1/0/2     | —               | —               | Trunk → SW-ACESSO-A (VLANs 10,30,31,99) |
| SW-CORE-L3        | G1/0/3     | —               | —               | Trunk → SW-ACESSO-B (VLANs 20,40,50,99) |

---

## SVIs do SW-CORE-L3 (Gateways Inter-VLAN)

| VLAN | Nome        | SVI / Gateway   | Rede             | DHCP       |
|------|-------------|-----------------|------------------|------------|
| 10   | OPERACIONAL | 192.168.10.1    | 192.168.10.0/24  | Sim (relay)|
| 20   | DIRETORIA   | 192.168.20.1    | 192.168.20.0/24  | Sim (relay)|
| 30   | SERVIDORES  | 192.168.30.1    | 192.168.30.0/24  | Não        |
| 31   | IMPRESSORAS | 192.168.31.1    | 192.168.31.0/24  | Não        |
| 40   | IoT         | 192.168.40.1    | 192.168.40.0/24  | Sim (relay)|
| 50   | GUEST       | 172.16.50.1     | 172.16.50.0/24   | Sim (relay)|
| 99   | MGMT/TI     | 10.0.99.1       | 10.0.99.0/24     | Não        |

> O roteador (`RTR-AGENCIA-BORDA`) atua como servidor DHCP centralizado para as VLANs 10, 20, 40 e 50. As SVIs do Core usam `ip helper-address 10.0.0.1` para relay das requisições DHCP.

---

## Dispositivos por VLAN

### VLAN 10 – Operacional (192.168.10.0/24)

| Hostname    | IP             | Interface SW-A | Atribuição |
|-------------|----------------|----------------|------------|
| V10-OP-01   | 192.168.10.11  | Fa0/1          | DHCP       |
| V10-OP-02   | 192.168.10.12  | Fa0/2          | DHCP       |
| *(demais)*  | 192.168.10.x   | Fa0/3–10       | DHCP       |

---

### VLAN 20 – Diretoria (192.168.20.0/24)

| Hostname    | IP             | Interface SW-B | Atribuição |
|-------------|----------------|----------------|------------|
| V20-DIR-01  | 192.168.20.11  | Fa0/1          | DHCP       |
| V20-DIR-02  | 192.168.20.12  | Fa0/2          | DHCP       |
| *(demais)*  | 192.168.20.x   | Fa0/3–5        | DHCP       |

---

### VLAN 30 – Servidores (192.168.30.0/24)

| Hostname    | IP             | Interface SW-A | Atribuição |
|-------------|----------------|----------------|------------|
| SRV-WEB-01  | 192.168.30.9   | Fa0/19         | Estática   |
| SRV-NAS-01  | 192.168.30.10  | Fa0/20         | Estática   |

---

### VLAN 31 – Impressoras (192.168.31.0/24)

| Hostname    | IP             | Interface SW-A | Atribuição |
|-------------|----------------|----------------|------------|
| PRT-CORP-01 | 192.168.31.10  | Fa0/21         | Estática   |

---

### VLAN 40 – IoT (192.168.40.0/24)

| Hostname    | IP             | Interface SW-B | Atribuição |
|-------------|----------------|----------------|------------|
| IOT-CAM-1   | 192.168.40.11  | Fa0/20         | DHCP       |
| IOT-CAM-2   | 192.168.40.12  | Fa0/19         | DHCP       |
| *(demais)*  | 192.168.40.x   | Fa0/18–24      | DHCP       |

---

### VLAN 50 – Guest / Wi-Fi (172.16.50.0/24)

| Hostname   | IP            | Interface SW-B | Atribuição |
|------------|---------------|----------------|------------|
| Visitante1 | 172.16.50.11  | G0/2 (AP)      | DHCP       |
| Visitante2 | 172.16.50.13  | G0/2 (AP)      | DHCP       |

> Conectividade via Access Point (AP) genérico ligado à G0/2 do SW-ACESSO-B.

---

### VLAN 99 – MGMT/TI (10.0.99.0/24)

| Hostname    | IP           | Interface SW-A | Atribuição |
|-------------|--------------|----------------|------------|
| MGMT-TI-01  | 10.0.99.50   | Fa0/24         | Estática   |
| SW-ACESSO-A | 10.0.99.10   | SVI VLAN 99    | Estática   |
| SW-ACESSO-B | 10.0.99.11   | SVI VLAN 99    | Estática   |

---

## Mapeamento de Portas por Switch

### SW-ACESSO-A (Catalyst 2960)

| Interface   | VLAN | Modo   | Descrição                  |
|-------------|------|--------|----------------------------|
| G0/1        | —    | Trunk  | Uplink → SW-CORE-L3        |
| Fa0/1–10    | 10   | Access | PCs Operacional            |
| Fa0/11–18   | —    | —      | Livres / Desativadas       |
| Fa0/19      | 30   | Access | SRV-WEB-01                 |
| Fa0/20      | 30   | Access | SRV-NAS-01                 |
| Fa0/21      | 31   | Access | PRT-CORP-01                |
| Fa0/22–24   | 99   | Access | PCs TI / Gerenciamento     |
| G0/2        | —    | —      | Livre / Desativada         |

### SW-ACESSO-B (Catalyst 2960)

| Interface   | VLAN | Modo   | Descrição                  |
|-------------|------|--------|----------------------------|
| G0/1        | —    | Trunk  | Uplink → SW-CORE-L3        |
| G0/2        | 50   | Access | Access Point (Wi-Fi Guest) |
| Fa0/1–5     | 20   | Access | Laptops Diretoria          |
| Fa0/6–17    | —    | —      | Livres / Desativadas       |
| Fa0/18–24   | 40   | Access | Câmeras / Sensores IoT     |

### SW-CORE-L3 (Catalyst 3650)

| Interface   | Descrição                        |
|-------------|----------------------------------|
| G1/0/1      | Link → RTR-AGENCIA-BORDA         |
| G1/0/2      | Trunk → SW-ACESSO-A              |
| G1/0/3      | Trunk → SW-ACESSO-B              |

### RTR-AGENCIA-BORDA (ISR 4321)

| Interface   | Descrição                        |
|-------------|----------------------------------|
| G0/0/0      | WAN – Modem ISP                  |
| G0/0/1      | LAN – Link → SW-CORE-L3          |
