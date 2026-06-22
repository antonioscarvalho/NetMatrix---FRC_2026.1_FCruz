# DNS — NetMatrix

**Servidor:** `192.168.10.10`  
**Software:** `bind9`  
**Responsável:** Luana Ribeiro

## Como subir o serviço

```bash
sudo apt install bind9 -y
sudo cp config/named.conf.local /etc/bind/named.conf.local
sudo cp config/db.netmatrix /etc/bind/db.netmatrix
sudo cp config/db.reverso /etc/bind/db.reverso
sudo systemctl restart bind9
sudo systemctl status bind9
```

## Testar resolução

```bash
host www.netmatrix.com.br 192.168.10.10
nslookup mail.netmatrix.com.br 192.168.10.10
```
