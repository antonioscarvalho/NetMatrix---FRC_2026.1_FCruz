# Questões — Laboratório DHCP

## Questão 1
> Coloque um analisador de tráfego (Wireshark/tcpdump) para identificar os diálogos entre cliente e servidor DHCP.

**Resposta:**

Com `tcpdump` capturando o tráfego entre o cliente e o servidor, a sequência observada foi o clássico **DORA**. Primeiro, o cliente, ainda sem IP (origem `0.0.0.0`), envia um **DHCPDISCOVER** em broadcast (`255.255.255.255:67`) perguntando se existe algum servidor DHCP disponível na rede. O servidor responde com um **DHCPOFFER**, também em broadcast (já que o cliente ainda não tem IP configurado), oferecendo um endereço (`192.168.10.150`) junto com as opções de rede — máscara, gateway, servidor DNS e tempo de lease. O cliente então confirma, em broadcast, que aceita aquela oferta especificamente através de um **DHCPREQUEST** (que inclui `Server-ID` e `Requested-IP`), o que também avisa qualquer outro servidor que tenha respondido que sua oferta não foi escolhida. Por fim, o servidor confirma a concessão definitiva do endereço com um **DHCPACK**, e a partir daí o cliente configura sua interface com o IP recebido.

## Questão 2
> Faça o servidor DHCP oferecer IPs apenas para MACs previamente cadastrados.

**Resposta:**

Isso foi feito removendo o pool dinâmico da subnet e deixando apenas reservas por endereço MAC (bloco `reservations`, equivalente ao `host {...}` do `dhcpd.conf` clássico). Com essa configuração, uma estação com MAC previamente cadastrado recebe sempre o mesmo IP reservado a ela. Já uma estação com MAC não cadastrado tem o seu `DISCOVER` recebido normalmente pelo servidor, mas nenhuma oferta é gerada para ela — sem nenhuma faixa dinâmica disponível e sem reserva correspondente ao seu MAC, o servidor simplesmente não tem endereço para oferecer, e a solicitação fica sem resposta.

## Questão 3
> Se colocarmos mais de um servidor DHCP na rede, o que pode acontecer?

**Resposta:**

Na prática, os dois servidores responderam de forma independente ao mesmo `DISCOVER`, cada um oferecendo um IP da sua própria faixa de endereços; o cliente aceitou a primeira oferta que chegou e ignorou a segunda, que chegou poucos segundos depois. Isso mostra que não existe nenhuma coordenação entre servidores DHCP independentes: cada um responde por conta própria a qualquer broadcast que receber, e quem "ganha" o cliente é, na prática, uma corrida pelo primeiro a responder. Se as faixas de endereço dos dois servidores se sobrepuserem, existe risco real de dois servidores atribuírem o mesmo IP a clientes diferentes ao longo do tempo, causando conflito de endereço na rede; e mesmo sem sobreposição, um servidor DHCP "não planejado" (rogue) é considerado um problema de segurança em redes reais, porque pode distribuir gateway e DNS forjados e desviar o tráfego dos clientes. Por isso, em redes corporativas a forma correta de ter redundância é com servidores que compartilham o mesmo banco de leases (failover DHCP coordenado), e não dois servidores independentes respondendo na mesma rede.
