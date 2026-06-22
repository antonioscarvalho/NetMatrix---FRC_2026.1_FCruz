# DHCP — NetMatrix

**Servidor:** `192.168.10.2`  
**Software:** `kea` (`kea-dhcp4`)  
**Responsável:** Maykon Soares

## Como subir o serviço

```bash
sudo dnf install kea -y
sudo cp config/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf
sudo systemctl restart kea-dhcp4
sudo systemctl status kea-dhcp4
```

> No roteiro original (Ubuntu/Debian) o equivalente é o `isc-dhcp-server` (`dhcpd3`), descontinuado no Fedora — ver `docs/relatorio.md` para os detalhes da adaptação.
