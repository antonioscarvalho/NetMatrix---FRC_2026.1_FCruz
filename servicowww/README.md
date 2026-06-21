# WWW — NetMatrix

**Servidor:** `192.168.10.30`  
**Software:** `apache2`  
**Responsável:** [nome do membro]

## Como subir o serviço

```bash
sudo apt install apache2 -y

# Copiar página principal
sudo cp html/grupo/index.html /var/www/html/index.html

# Criar diretórios dos domínios virtuais
for membro in antonio luana cristiano maykon; do
  sudo mkdir -p /var/www/$membro.netmatrix.com.br
  sudo cp html/$membro/index.html /var/www/$membro.netmatrix.com.br/index.html
done

# Ativar virtual hosts
for membro in antonio luana cristiano maykon; do
  sudo cp config/$membro.netmatrix.com.br.conf /etc/apache2/sites-available/
  sudo a2ensite $membro.netmatrix.com.br.conf
done

sudo a2enmod userdir include
sudo systemctl reload apache2
```

## Testar

```bash
curl http://www.netmatrix.com.br
curl http://antonio.netmatrix.com.br
```
