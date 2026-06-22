# Questões — Laboratório WWW

## Questão 1

> Publique páginas gerais e páginas por usuário via public_html.

**Resposta:** Para habilitar páginas por usuário no Apache, ativamos o módulo `userdir` executando o comando `sudo a2enmod userdir` e reiniciamos o serviço com `sudo systemctl restart apache2`. Isso permite que cada usuário do sistema operacional possua uma pasta chamada `public_html` em seu diretório `/home`. Os arquivos estáticos de uso geral continuam hospedados no diretório principal (`/var/www/html`), enquanto a página individual de cada usuário pode ser acessada pela URL do servidor adicionando um til antes do nome, como `http://www.netmatrix.com.br/~antonio`.

## Questão 2

> Controle de acesso com .htaccess.

**Resposta:** Para aplicar restrições de acesso a diretórios específicos, editamos o arquivo de configuração principal (`/etc/apache2/apache2.conf`)  e alteramos a diretiva de `AllowOverride None` para `AllowOverride All` no bloco correspondente ao diretório alvo (como `/home/*/public_html`). Em seguida, criamos um arquivo chamado `.htaccess` na pasta do usuário contendo regras de autenticação (`AuthType Basic`, `AuthName`, `AuthUserFile`) para exigir validação de senha. O arquivo que armazena as credenciais é gerado no Linux utilizando o utilitário `htpasswd`, garantindo que o diretório só possa ser escrito pelo próprio dono e lido pelos demais mediante controle.

## Questão 3

> Domínios virtuais — um por aluno.

**Resposta:** A implementação de domínios virtuais (*Virtual Hosts*) permite hospedar múltiplos sites utilizando um único endereço IP. Dentro do diretório `/etc/apache2/sites-available`, criamos arquivos de configuração (`.conf`) específicos para cada aluno. Nesses arquivos, utilizamos a diretiva `<VirtualHost *:80>`, definindo o parâmetro `ServerName` para o subdomínio correspondente (exemplo: `www.antonio.netmatrix.com.br`) e o `DocumentRoot` apontando para a pasta individual do aluno. Por fim, ativamos os sites com o comando `a2ensite`, recarregamos o Apache e garantimos que o servidor DNS aponte esses domínios para o IP do nosso servidor Web.

## Questão 4

> O que são SSI? Demonstre.

**Resposta:** SSI (Server Side Includes) é um mecanismo executado pelo servidor Web utilizado para inserir conteúdo dinâmico em páginas HTML estáticas pouco antes de serem enviadas ao cliente. Para habilitar e demonstrar essa funcionalidade, ativamos o módulo correspondente com `a2enmod include` e configuramos a diretiva `Options +Includes` para extensões específicas (como `.shtml`). Quando o Apache processa um arquivo com essa configuração, ele identifica comandos embutidos no código, como ``, e os substitui por dados reais (neste caso, a data e hora do sistema) em tempo de execução.

## Questão 5

> Mecanismos de concorrência do Apache (MPMs).

**Resposta:** O Apache gerencia múltiplas conexões simultâneas através dos MPMs (*Multi-Processing Modules*). Os principais modelos são: o **Prefork** (que aloca um processo filho dedicado para cada conexão, garantindo alta estabilidade), o **Worker** (que utiliza processos contendo múltiplas threads, reduzindo drasticamente o consumo de memória) e o **Event** (que delega as conexões ativas do tipo *Keep-Alive* a uma thread dedicada, liberando as demais para processar novas requisições). O ajuste desses mecanismos é feito através de diretivas de configuração em `/etc/apache2/`, como `MaxRequestWorkers` e `KeepAliveTimeout`.

## Questão 6

> Como funciona um proxy server HTTP?

**Resposta:** Um servidor proxy HTTP funciona como um intermediário entre o cliente (navegador) e o servidor Web de destino. Ele intercepta as requisições HTTP do usuário e as repassa para a internet. Suas principais utilidades incluem o **caching** (armazenamento de páginas frequentemente acessadas para acelerar o carregamento e economizar largura de banda), a filtragem de conteúdo e segurança (bloqueio de domínios), ou a atuação como **Proxy Reverso**, onde o servidor recebe conexões da internet e as distribui para balancear a carga entre servidores Web internos.

## Questão 7

> Relação entre MIME e o serviço Web.

**Resposta:** O protocolo MIME (*Multipurpose Internet Mail Extensions*) é o padrão responsável por identificar e classificar os tipos de arquivos transmitidos na internet. A relação direta com o serviço Web ocorre porque o protocolo HTTP não analisa a extensão do arquivo (como `.html` ou `.jpg`) para instruir o navegador. Em vez disso, o Apache consulta o arquivo de mapeamento `/etc/mime.types` e envia no cabeçalho da resposta HTTP o campo `Content-Type` (por exemplo, `text/html` ou `image/png`). É essa informação MIME que determina como o navegador do cliente irá renderizar ou processar o conteúdo recebido.

## Questão 8

> Conexão telnet no servidor Web com GET e HEAD.

**Resposta:**
Utilizando o comando `telnet 127.0.0.1 80` no terminal, emulamos o comportamento de um cliente HTTP.

Ao enviarmos a requisição:

```
GET / HTTP/1.1
Host: www.netmatrix.com.br


```

O servidor retornou o código de status `HTTP/1.1 200 OK`, seguido do cabeçalho completo (contendo o `Content-Type`, data, servidor `Apache/2.4`) e todo o código-fonte HTML da página principal no corpo da mensagem.

Ao enviarmos a requisição:

```
HEAD / HTTP/1.1
Host: www.netmatrix.com.br


```

O servidor retornou exatamente os mesmos cabeçalhos da requisição GET (incluindo status e metadados da página), mas omitiu completamente o corpo (o código HTML), demonstrando a utilidade do método HEAD para verificar informações de um recurso sem transferir o arquivo inteiro.

## Questão 9

> Página HTML principal com dados do grupo e links para domínios virtuais.

**Resposta:** Ver `/var/www/html/index.html` (Conforme imagem do *front-end* e mapa de rede da intranet NetMatrix apresentados no projeto prático).