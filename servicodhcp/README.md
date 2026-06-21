# DHCP — NetMatrix

**Servidor:** `192.168.10.1`  
**Software:** `isc-dhcp-server`  
**Responsável:** [nome do membro]

## Como subir o serviço

```bash
sudo apt install isc-dhcp-server -y
sudo cp config/dhcpd.conf /etc/dhcp/dhcpd.conf
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
