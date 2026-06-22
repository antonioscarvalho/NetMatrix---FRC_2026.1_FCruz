# Serviço DNS — NetMatrix
**Responsável:** Luana Ribeiro

## Configuração realizada
Foi implementado um servidor DNS autoritativo para o domínio **netmatrix.com.br**, usando o software **BIND 9** (pacote `bind`/`bind-utils`), responsável por:

- **Zona direta** `netmatrix.com.br`: traduz nomes de máquinas da empresa para endereços IP (registros `A`), além de declarar o servidor de nomes (`NS`) e o servidor de e-mail (`MX`).
- **Zona reversa** `10.168.192.in-addr.arpa`: traduz endereços IP da rede `192.168.10.0/24` de volta para nomes (registros `PTR`).

### Mapa de endereços da intranet (parte referente ao DNS)

| Nome (FQDN)                  | IP             | Papel                          |
|-------------------------------|----------------|---------------------------------|
| ns1.netmatrix.com.br          | 192.168.10.1   | Servidor DNS (este servidor)   |
| dhcp.netmatrix.com.br         | 192.168.10.2   | Servidor DHCP                  |
| mail.netmatrix.com.br         | 192.168.10.3   | Servidor de e-mail (SMTP/POP3) |
| www.netmatrix.com.br          | 192.168.10.4   | Servidor Web                    |
| antonio.netmatrix.com.br      | 192.168.10.101 | Estação — Antonio Carvalho     |
| cristino.netmatrix.com.br     | 192.168.10.102 | Estação — Cristino             |
| luana.netmatrix.com.br        | 192.168.10.103 | Estação — Luana Ribeiro        |
| maykon.netmatrix.com.br       | 192.168.10.104 | Estação — Maykon Soares        |

*(IPs dos servidores DHCP/Mail/WWW são uma proposta para manter o mapa de nomes coerente; devem ser confirmados com quem configurou cada serviço.)*

### Ambiente usado e adaptações em relação ao roteiro original

A configuração real foi feita em **Ubuntu/Debian**:

| Item do roteiro (Ubuntu/Debian)         | Equivalente usado no Fedora 42                 |
|------------------------------------------|--------------------------------------------------|
| pacote `bind9`                           | pacote `bind` + `bind-utils`                     |
| `/etc/bind/named.conf`                   | `/etc/named.conf`                                |
| `/etc/bind/named.conf.options`           | `/etc/named.conf.options` (criado manualmente, incluído via `include`) |
| `/etc/bind/named.conf.local`             | `/etc/named.conf.local` (criado manualmente, incluído via `include`) |
| `/etc/bind/db.*`                          | `/var/named/db.*`                                |
| `/etc/init.d/bind9 start`                 | `systemctl start named` (systemd)                |
| comando `dnsdomainname`                   | não disponível por padrão no Fedora 42 (utilitário legado); equivalente conceitual obtido via `hostnamectl` / inspeção do FQDN em `/etc/hosts` |

Por padrão, o Fedora concentra tudo em um único `/etc/named.conf`; recriamos a divisão em 3 arquivos (`named.conf` + `named.conf.options` + `named.conf.local`) só para manter a mesma organização didática do roteiro original.

**Observação sobre o ambiente de testes:** a configuração foi validada em uma máquina de desenvolvimento pessoal, não na máquina física do laboratório da disciplina. Para isolar o servidor DNS sem interferir na resolução de nomes real da máquina (usada para outras tarefas), foi criada uma interface de rede dedicada (`dns-lan`, tipo *dummy*) com o endereço `192.168.10.1/24`, e o BIND foi configurado para escutar nela. O `/etc/resolv.conf` e o `/etc/hosts` globais do sistema **não foram sobrescritos** (são regenerados automaticamente pelo WSL a cada inicialização); em vez disso, a resolução foi testada apontando explicitamente para o servidor (`dig/host/nslookup @192.168.10.1`), e as entradas de exemplo que uma estação real da intranet usaria estão documentadas abaixo e no arquivo `config/resolv.conf.exemplo`. No laboratório físico da disciplina (Ubuntu), o procedimento é o mesmo do roteiro original, sem essa ressalva.

---

## 2. Arquivos de configuração

Todos os arquivos abaixo estão em `config/` e foram efetivamente aplicados e testados.

### `named.conf`
```
include "/etc/named.conf.options";
include "/etc/named.conf.local";
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### `named.conf.options`
```
options {
    listen-on port 53 { 127.0.0.1; 192.168.10.1; };
    listen-on-v6 port 53 { none; };
    directory       "/var/named";
    allow-query     { any; };
    allow-recursion { 192.168.10.0/24; 127.0.0.1; };
    recursion yes;
    forwarders { 8.8.8.8; 1.1.1.1; };
    dnssec-validation no;
};
```
*(arquivo completo em `config/named.conf.options`)*

### `named.conf.local` — declaração das zonas
```
zone "netmatrix.com.br" IN {
    type master;
    file "db.netmatrix.com.br";
    allow-update { none; };
};

zone "10.168.192.in-addr.arpa" IN {
    type master;
    file "db.192.168.10";
    allow-update { none; };
};
```

### `db.netmatrix.com.br` — zona direta
```
$TTL    604800
@       IN      SOA     ns1.netmatrix.com.br. admin.netmatrix.com.br. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Minimo
;
@       IN      NS      ns1.netmatrix.com.br.
@       IN      MX      10 mail.netmatrix.com.br.

ns1       IN      A       192.168.10.1
dhcp      IN      A       192.168.10.2
mail      IN      A       192.168.10.3
www       IN      A       192.168.10.4
@         IN      A       192.168.10.4

antonio   IN      A       192.168.10.101
cristino  IN      A       192.168.10.102
luana     IN      A       192.168.10.103
maykon    IN      A       192.168.10.104
```

### `db.192.168.10` — zona reversa
```
$TTL    604800
@       IN      SOA     ns1.netmatrix.com.br. admin.netmatrix.com.br. (
                              1
                         604800
                          86400
                        2419200
                         604800 )
;
@       IN      NS      ns1.netmatrix.com.br.

1       IN      PTR     ns1.netmatrix.com.br.
2       IN      PTR     dhcp.netmatrix.com.br.
3       IN      PTR     mail.netmatrix.com.br.
4       IN      PTR     www.netmatrix.com.br.
101     IN      PTR     antonio.netmatrix.com.br.
102     IN      PTR     cristino.netmatrix.com.br.
103     IN      PTR     luana.netmatrix.com.br.
104     IN      PTR     maykon.netmatrix.com.br.
```

### `resolv.conf.exemplo` — como ficaria em uma estação cliente real
```
domain netmatrix.com.br
nameserver 192.168.10.1
```

---

## 3. Testes realizados (evidências em `evidencias/`)

Validação de sintaxe:
```
$ sudo named-checkconf /etc/named.conf
$ sudo named-checkzone netmatrix.com.br /var/named/db.netmatrix.com.br
zone netmatrix.com.br/IN: loaded serial 1
OK
$ sudo named-checkzone 10.168.192.in-addr.arpa /var/named/db.192.168.10
zone 10.168.192.in-addr.arpa/IN: loaded serial 1
OK
```

Inicialização manual em modo debug (equivalente ao `named -f -g -d 1` do roteiro original) — log completo em `evidencias/named_debug_output_resumo.log`:
```
running as: named -f -g -d 1 -u named -c /etc/named.conf
listening on IPv4 interface lo, 127.0.0.1#53
listening on IPv4 interface dns-lan, 192.168.10.1#53
zone netmatrix.com.br/IN: loaded serial 1
zone 10.168.192.in-addr.arpa/IN: loaded serial 1
```

Bateria de testes de resolução (`dig`, `host -d`, `nslookup` interativo) — saída completa em `evidencias/testes_dns.txt`. Destaques:

- **Consulta direta (A):** `maykon.netmatrix.com.br` → `192.168.10.104`
- **Consulta reversa (PTR):** `192.168.10.104` → `maykon.netmatrix.com.br`
- **Consulta NS:** `netmatrix.com.br` → `ns1.netmatrix.com.br`
- **Consulta MX:** `netmatrix.com.br` → `10 mail.netmatrix.com.br`
- **Consulta SOA:** serial, refresh, retry, expire e TTL mínimo conferem com a zona
- **Autoritativa vs. não-autoritativa:** consulta a `netmatrix.com.br` retorna flag `aa` (nosso servidor é dono da zona); consulta a `wikipedia.org` (resolvida via *forwarders*) **não** retorna `aa` — resposta obtida de terceiros, não autoritativa
- **`nslookup` interativo:** troca de servidor (`server 192.168.10.1` → `server 8.8.8.8`), `set type=ptr`, `set type=mx`; ao consultar `8.8.8.8` por `netmatrix.com.br`, o retorno é `NXDOMAIN` — esperado, pois é um domínio fictício, que só existe no nosso servidor interno, não no DNS público

---

