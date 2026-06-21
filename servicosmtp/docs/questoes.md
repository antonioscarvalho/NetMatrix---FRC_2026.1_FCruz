# Questões — Laboratório SMTP

## Questão 1
> Descreva as principais modificações em `/etc/postfix/main.cf` e `master.cf`.

**Resposta:**

## Questão 2
> Qual a diferença entre mbox e maildir?

**Resposta:**

## Questão 3
> Para que servem os esquemas SASL/TLS?

**Resposta:**

## Questão 4
> Qual a diferença entre RFC822 e MIME types?

**Resposta:**

## Questão 5
> Faça uma conexão telnet no servidor SMTP e envie um e-mail com os comandos do protocolo.

**Resposta:**
```
HELO netmatrix.com.br
MAIL FROM: <antonio@netmatrix.com.br>
RCPT TO: <luana@netmatrix.com.br>
DATA
Subject: Teste
.
QUIT
```

## Questão 6
> Faça uma conexão telnet no servidor POP e receba e-mails.

**Resposta:**
```
USER luana
PASS senha
LIST
RETR 1
QUIT
```
