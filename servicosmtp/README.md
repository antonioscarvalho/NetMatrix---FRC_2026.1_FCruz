# SMTP — NetMatrix

**Servidor:** `192.168.10.20`  
**Software:** `postfix` + `dovecot`  
**Responsável:** [nome do membro]

## Como subir o serviço

```bash
sudo apt install postfix dovecot-pop3d -y
sudo cp config/main.cf /etc/postfix/main.cf
sudo systemctl restart postfix dovecot
sudo systemctl status postfix
```

## Testar envio

```bash
mail -v antonio@netmatrix.com.br
```
