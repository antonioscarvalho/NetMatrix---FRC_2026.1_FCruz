# 🌐 NetMatrix — Intranet Lab (FRC 2026.1)

> Trabalho prático da disciplina **Fundamentos de Redes de Computadores**  
> UnB/FCTE — Engenharia de Software | Prof. Fernando W. Cruz | Turma 2026.1

---

## 🏢 A Empresa

**NetMatrix** é uma empresa fictícia criada para simular a infraestrutura de uma intranet corporativa real, com os principais serviços de rede configurados e operando em ambiente Linux/Ubuntu.

---

## 👥 Equipe

| Nome | E-mail corporativo |
|------|-------------------|
| Antonio Carvalho | antonio.carvalho@netmatrix.com.br |
| Luana Ribeiro | luana.ribeiro@netmatrix.com.br |
| Cristiano Morais | cristiano.morais@netmatrix.com.br |
| Maykon.Soares | maykon.soares@netmatrix.com.br |

---

## 🌍 Configurações de Rede

| Parâmetro | Valor |
|-----------|-------|
| **Domínio** | `netmatrix.com.br` |
| **Classe de endereçamento** | Classe C |
| **Rede** | `192.168.10.0/24` |
| **Máscara de sub-rede** | `255.255.255.0` |
| **Broadcast** | `192.168.10.255` |
| **Gateway/Roteador** | `192.168.10.1` |
| **Faixa dinâmica (DHCP)** | `192.168.10.100 – 192.168.10.200` |

---

## 🖥️ Servidores da Intranet

| Hostname | IP | Serviço |
|----------|----|---------|
| `dns.netmatrix.com.br` | `192.168.10.10` | DNS (BIND9) |
| `mail.netmatrix.com.br` | `192.168.10.20` | SMTP (Postfix) + POP3 (Dovecot) |
| `www.netmatrix.com.br` | `192.168.10.30` | WWW (Apache2) |
| `dhcp.netmatrix.com.br` | `192.168.10.1` | DHCP (isc-dhcp-server) |

> ⚠️ IPs e hostnames sujeitos a alteração conforme andamento do laboratório. Este README será atualizado.

---

## 🛠️ Serviços Implementados

### 🔹 DHCP — Dynamic Host Configuration Protocol
- Servidor: `isc-dhcp-server`
- Distribui endereços IP automaticamente na faixa `192.168.10.100–200`
- Suporta reserva de IP por endereço MAC

### 🔹 DNS — Domain Name System
- Servidor: `BIND9`
- Zona direta: `netmatrix.com.br`
- Zona reversa: `10.168.192.in-addr.arpa`
- Registro MX apontando para `mail.netmatrix.com.br`

### 🔹 SMTP — Simple Mail Transfer Protocol
- Servidor: `Postfix`
- Acesso a e-mails via POP3: `Dovecot`
- Domínio de e-mail: `@netmatrix.com.br`

### 🔹 WWW — Servidor Web
- Servidor: `Apache2`
- Página principal do grupo em `http://www.netmatrix.com.br`
- Domínios virtuais para cada membro da equipe
- Relatório do trabalho publicado na página principal

---

## 📁 Estrutura do Repositório

```
netmatrix-frc/
├── README.md
├── dhcp/
│   ├── dhcpd.conf
│   └── relatorio-dhcp.md
├── dns/
│   ├── named.conf.local
│   ├── db.netmatrix
│   ├── db.reverso
│   └── relatorio-dns.md
├── smtp/
│   ├── main.cf
│   ├── master.cf
│   └── relatorio-smtp.md
└── www/
    ├── netmatrix.conf
    ├── html/
    │   └── index.html
    └── relatorio-www.md
```

---

## 🚀 Como Reproduzir o Ambiente

### Pré-requisitos
- VirtualBox (ou similar)
- Ubuntu Server 22.04 LTS
- Tailscale (para conectar as VMs remotamente)

### Passos gerais

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/netmatrix-frc.git

# 2. Configure cada serviço seguindo o roteiro na pasta correspondente
cd netmatrix-frc/dns
# Siga as instruções em relatorio-dns.md

# 3. Reinicie os serviços após alterações
sudo systemctl restart bind9      # DNS
sudo systemctl restart isc-dhcp-server  # DHCP
sudo systemctl restart postfix    # SMTP
sudo systemctl restart apache2    # WWW
```

---

## 📋 Referências

- TANENBAUM, A. *Computer Networks*. 3ª Edição, Prentice Hall, 1996.
- Roteiros de laboratório: DHCP, DNS, SMTP e WWW — Prof. Fernando W. Cruz
- [Postfix Documentation](https://www.postfix.org/)
- [BIND9 Documentation](https://bind9.readthedocs.io/)
- [Apache2 Documentation](https://httpd.apache.org/docs/)
- [NAT HOWTO](https://www.netfilter.org/documentation/HOWTO/pt/NAT-HOWTO-6.html)

---

*Entrega: 19/06/2026 — NetMatrix © 2026*
