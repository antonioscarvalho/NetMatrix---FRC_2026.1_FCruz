# Questões — Laboratório DNS

## Questão 1
> Qual é o retorno do comando `dnsdomainname`? O que significa?

**Resposta:**

`dnsdomainname` retorna apenas a parte de domínio do FQDN da máquina (tudo depois do nome curto do host) — por exemplo, para uma máquina `maykon.netmatrix.com.br`, o comando retornaria `netmatrix.com.br`. Esse valor normalmente é obtido a partir da entrada do próprio host em `/etc/hosts` (ou de um domínio NIS) e é o mesmo valor usado na diretiva `domain` do `/etc/resolv.conf` — ele é anexado automaticamente quando o usuário digita um nome curto (não qualificado) em uma consulta. No ambiente de testes usado (Fedora 42), esse utilitário legado não está mais empacotado por padrão (foi substituído por `hostnamectl`/`nmcli`); no laboratório físico (Ubuntu), o comando funciona como descrito no roteiro.

## Questão 2
> O que é o nome localhost? E o endereço 127.0.0.1?

**Resposta:**

`localhost` é o nome convencional que toda máquina usa para se referir a si mesma. `127.0.0.1` é o endereço IPv4 de *loopback*: qualquer pacote enviado a ele nunca sai da própria máquina, sendo devolvido pela própria pilha de rede do sistema operacional. Esse par nome/endereço deve sempre existir porque diversos serviços, scripts e bibliotecas do sistema assumem poder se conectar a si mesmos via rede (TCP/IP) mesmo sem nenhuma interface de rede física ativa ou qualquer servidor DNS disponível — por exemplo, bancos de dados locais, servidores web em desenvolvimento, ferramentas de diagnóstico, X11 forwarding, etc. Sem essa entrada, a resolução de `localhost` dependeria de DNS externo (lento ou indisponível) ou simplesmente falharia.

## Questão 3
> O que é o FQDN?

**Resposta:**

FQDN (*Fully Qualified Domain Name*, "Nome de Domínio Totalmente Qualificado") é o nome completo e absoluto de uma máquina dentro da hierarquia DNS, contendo o nome do host **mais** todos os domínios pais até a raiz — por exemplo `maykon.netmatrix.com.br.` (o ponto final representa a raiz da hierarquia). Diferente do nome curto (`maykon`), que só é único dentro de um domínio específico, o FQDN identifica a máquina de forma única em toda a Internet/intranet.

## Questão 4
> Podemos ter 2 servidores DNS na mesma rede? Qual a configuração mais adequada?

**Resposta:**

Sim — na verdade, ter pelo menos dois servidores é a prática recomendada para disponibilidade. A configuração adequada é o modelo **master/slave (primário/secundário)**: o servidor *master* mantém a cópia oficial (editável) da zona; um ou mais servidores *slave* fazem periodicamente uma transferência de zona (AXFR/IXFR) a partir do master, copiando os mesmos dados, e respondem às consultas dos clientes de forma idêntica (porém somente leitura — qualquer alteração deve ser feita no master). O ritmo dessas transferências é controlado pelos parâmetros do registro `SOA` (*refresh*, *retry*, *expire*). Do lado do cliente, configuram-se as duas entradas `nameserver` no `resolv.conf` (como no exemplo do próprio roteiro), e o resolver tenta o primeiro e cai para o segundo se o primeiro não responder. Na NetMatrix, isso seria implementado adicionando uma segunda máquina com `type slave; masters { 192.168.10.1; };` para as mesmas zonas.

## Questão 5
> O que é DNS reverso? Como foi implementado no lab?

**Resposta:**

DNS reverso é o processo inverso da resolução normal: em vez de traduzir nome → IP, ele traduz **IP → nome**, usando registros do tipo `PTR`. Para isso existe uma zona especial para cada bloco de rede, sob o domínio `in-addr.arpa`, com os octetos do endereço de rede escritos em **ordem invertida**. No laboratório, para a rede `192.168.10.0/24`, foi criada a zona `10.168.192.in-addr.arpa` (arquivo `db.192.168.10`), com um registro `PTR` para cada host — por exemplo, o IP `192.168.10.104` aponta para `maykon.netmatrix.com.br`. Isso foi testado com `dig -x 192.168.10.104` e `host -d 192.168.10.4`, confirmando a tradução correta. DNS reverso é usado, por exemplo, por servidores de e-mail para validar a origem de mensagens (anti-spam) e em logs/auditoria de segurança, para identificar a máquina de origem de uma conexão.

## Questão 6
> O que é a entrada MX? Podem haver mais de uma?

**Resposta:**

O registro `MX` (*Mail Exchanger*) indica qual(is) servidor(es) são responsáveis por receber e-mails endereçados ao domínio — é a informação que um servidor SMTP remetente consulta para saber para onde entregar uma mensagem destinada a `alguem@netmatrix.com.br`. Sim, pode haver mais de um registro MX para o mesmo domínio, cada um com um valor de prioridade (quanto **menor** o número, maior a prioridade); o remetente tenta primeiro o servidor de maior prioridade e usa os demais como contingência, caso o preferido esteja indisponível. Na zona criada, há um único registro (`@ IN MX 10 mail.netmatrix.com.br.`); um segundo servidor de e-mail de backup poderia ser adicionado, por exemplo, como `@ IN MX 20 mail2.netmatrix.com.br.`.

## Questão 7
> O que é resposta autoritativa?

**Resposta:**

Uma resposta é **autoritativa** quando vem do servidor que efetivamente possui a cópia oficial (master ou slave) da zona consultada — ou seja, a "fonte da verdade" para aquele domínio, e não uma cópia armazenada temporariamente em cache a partir de outra consulta. Nas ferramentas de teste (`dig`, `host`), isso é indicado pela flag **`aa`** (*Authoritative Answer*) no cabeçalho da resposta. Isso foi demonstrado na prática: ao consultar nosso servidor sobre `netmatrix.com.br` (zona da qual ele é master), a resposta vem com a flag `aa`; ao consultar o mesmo servidor sobre `wikipedia.org` (fora da nossa zona), a resposta é obtida por recursão através dos *forwarders* configurados, e vem **sem** a flag `aa` — uma resposta não-autoritativa, de segunda mão.

## Questão 8
> O que é um servidor caching-only?

**Resposta:**

Um servidor *caching-only* é um servidor DNS que **não é autoritativo para nenhuma zona própria** — ele apenas recebe consultas recursivas dos clientes, busca as respostas junto a outros servidores (root, TLD, autoritativos) e **guarda essas respostas em cache** pelo tempo definido no TTL de cada registro, agilizando consultas repetidas. Sua configuração corresponde basicamente a um `named.conf` só com a seção `options{}` (e as zonas padrão de loopback/`localhost`), sem nenhuma declaração `zone "..." { type master; ... }` personalizada. É útil para reduzir latência e tráfego externo da rede sem que a organização precise hospedar zona alguma.
