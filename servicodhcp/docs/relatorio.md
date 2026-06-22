# Serviço DHCP — NetMatrix
**Responsável:** Maykon Soares

## Configuração realizada
Foi implementado um servidor **DHCP** para a intranet **netmatrix.com.br**, responsável por atribuir dinamicamente endereço IP, máscara, gateway, servidor DNS e nome de domínio às estações da rede `192.168.10.0/24`, além de reservar endereços fixos por endereço MAC para estações conhecidas (equivalente ao bloco `host {...}` do roteiro original).

O roteiro original (UnB/FCTE) assume Ubuntu/Debian com `isc-dhcp-server` (`dhcpd3`). Esse pacote "clássico" da ISC foi descontinuado no Fedora; o substituto oficial, também da ISC, é o **Kea**. Os conceitos são os mesmos (subnet, pool de endereços, opções de rede, lease time, reserva por MAC) — só a sintaxe de configuração muda (JSON em vez do `dhcpd.conf` tradicional).

### Ambiente usado e adaptações em relação ao roteiro original

| Item | Ubuntu/Debian (roteiro original) | Fedora (usado aqui) |
|---|---|---|
| Servidor | `isc-dhcp-server` (`dhcpd3`) | `kea` (`kea-dhcp4`) |
| Cliente | `dhcp3-client` | `dhcp-client` (`dhclient` — mesmo pacote ISC, ainda mantido) |
| Arquivo de config | `/etc/dhcp3/dhcpd.conf` | `/etc/kea/kea-dhcp4.conf` (JSON) |
| Leases | `/var/lib/dhcp3/dhcpd.leases` | `/var/lib/kea/kea-leases4.csv` |
| Iniciar manualmente (debug) | `/usr/sbin/dhcpd3 -d -f <iface>` | `kea-dhcp4 -c <config>` (roda em foreground por padrão) |
| Serviço (systemd) | `dhcpd3-server` | `kea-dhcp4` |

### Mapa de endereços da intranet (parte referente ao DHCP)

| Nome (FQDN)               | IP             | Papel                                       |
|----------------------------|----------------|----------------------------------------------|
| ns1.netmatrix.com.br       | 192.168.10.1   | Servidor DNS                                  |
| dhcp.netmatrix.com.br      | 192.168.10.2   | Servidor DHCP (este servidor)                 |
| mail.netmatrix.com.br      | 192.168.10.3   | Servidor de e-mail                            |
| www.netmatrix.com.br       | 192.168.10.4   | Servidor Web                                  |
| maykon.netmatrix.com.br    | 192.168.10.104 | Estação — Maykon Soares (reserva por MAC)     |

Além dos hosts fixos acima, a faixa **192.168.10.150 – 192.168.10.200** é o pool dinâmico distribuído para as demais estações da rede.

**Observação sobre o ambiente de testes:** como as máquinas do grupo estão em locais diferentes, o servidor e os clientes de teste foram simulados localmente com Linux network namespaces, ligados por uma bridge isolada (`dhcp-br`), sem depender de rede física nem tocar na rede real da máquina. Por padrão, o WSL2 bloqueia o tráfego encaminhado entre portas de uma bridge criada manualmente (`iptables FORWARD` com política `DROP`); foi necessário liberar explicitamente com `sudo iptables -I FORWARD -i dhcp-br -o dhcp-br -j ACCEPT`. Diferente do que foi tentado para o serviço de DNS, esse teste não foi feito entre as máquinas físicas do grupo: o DHCP depende de **broadcast de camada 2** (`255.255.255.255`), que não atravessa VPNs roteadas como o Tailscale — diferente do DNS, que é só tráfego unicast comum. Testar DHCP de verdade entre máquinas físicas exigiria as máquinas estarem no mesmo segmento de rede local (mesmo switch/Wi-Fi).

---

## 2. Arquivos de configuração

Arquivo completo em `config/kea-dhcp4.conf`.

### `kea-dhcp4.conf`
```json
{
"Dhcp4": {
    "interfaces-config": {
        "interfaces": [ "dhcp-kea" ]
    },
    "control-socket": {
        "socket-type": "unix",
        "socket-name": "kea4-ctrl-socket"
    },
    "lease-database": {
        "type": "memfile",
        "lfc-interval": 3600
    },
    "valid-lifetime": 600,
    "min-valid-lifetime": 600,
    "max-valid-lifetime": 7200,
    "subnet4": [
        {
            "id": 1,
            "subnet": "192.168.10.0/24",
            "interface": "dhcp-kea",
            "pools": [
                { "pool": "192.168.10.150 - 192.168.10.200" }
            ],
            "option-data": [
                { "name": "routers", "data": "192.168.10.2" },
                { "name": "domain-name-servers", "data": "192.168.10.1" },
                { "name": "domain-name", "data": "netmatrix.com.br" },
                { "name": "broadcast-address", "data": "192.168.10.255" }
            ],
            "reservations": [
                {
                    "hw-address": "92:18:5d:fa:99:bc",
                    "ip-address": "192.168.10.104",
                    "hostname": "maykon"
                }
            ]
        }
    ]
}
}
```

Tempo de lease: **default 600 s (10 min)**, **máximo 7200 s (2 h)** — os mesmos valores do exemplo do roteiro original.

---

## 3. Testes realizados (evidências em `evidencias/`)

### Obtenção de lease (DORA completo)
Cliente real (`dhclient`) solicitando endereço, com `tcpdump` capturando a troca completa — evidências em `evidencias/dora_captura.pcap` / `dora_captura_dump.txt`:
```
DHCPDISCOVER on cliente-veth to 255.255.255.255 port 67
DHCPOFFER of 192.168.10.150 from 192.168.10.2
DHCPREQUEST for 192.168.10.150 on cliente-veth to 255.255.255.255 port 67
DHCPACK of 192.168.10.150 from 192.168.10.2
bound to 192.168.10.150 -- renewal in 288 seconds.
```
A captura mostra as 4 mensagens com todas as opções entregues: máscara, gateway, servidor DNS (192.168.10.1), domínio (`netmatrix.com.br`) e tempo de lease (600 s).

### Restrição por endereço MAC
Servidor reconfigurado sem pool dinâmico, apenas com reservas — só estações com MAC previamente cadastrado recebem endereço. O MAC conhecido (`92:18:5d:fa:99:bc`, reservado para "maykon") recebeu `192.168.10.104` normalmente; um MAC desconhecido teve o `DISCOVER` recebido pelo servidor, mas nenhuma oferta foi gerada — log em `evidencias/q2_mac_restrito.log`:
```
DHCP4_PACKET_RECEIVED ... DHCPDISCOVER (type 1) received from 0.0.0.0 ...
ALLOC_ENGINE_V4_ALLOC_FAIL_SUBNET ... failed to allocate an IPv4 lease ...
ALLOC_ENGINE_V4_ALLOC_FAIL_NO_POOLS ... no pools were available for the address allocation
ALLOC_ENGINE_V4_ALLOC_FAIL_CLASSES ... Failed to allocate an IPv4 address for client with classes: ALL, UNKNOWN
```
O cliente repetiu o `DISCOVER` várias vezes e nunca recebeu `OFFER` — exatamente o comportamento esperado.

### Dois servidores DHCP na mesma rede
Um segundo servidor (`dnsmasq`, atuando como servidor "não planejado"/rogue) foi colocado na mesma rede, com uma faixa de endereços diferente (192.168.10.160–180) — evidências em `evidencias/q3_dois_servidores.pcap`, `q3_dhclient_output.log` e `q3_dnsmasq_output.log`. Um cliente novo enviou um único `DISCOVER` e os dois servidores responderam: o Kea respondeu primeiro (`DHCPOFFER of 192.168.10.150 from 192.168.10.2`) e o cliente já tinha aceitado essa oferta (`REQUEST`/`ACK`) quando a oferta do segundo servidor chegou, cerca de 3 segundos depois — essa segunda oferta foi descartada pelo cliente.
