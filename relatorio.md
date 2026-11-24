# Relatório — Atividade Prática: Sockets TCP

**Disciplina:** Computação Distribuída (INE5418)
**Dupla:** Leonardo de Sousa Marques
**Data:** 24-11-2026

---

## Nível 1 — Inspecionar o código

### 1.1 Servidor (`servidor.py`)

Descreva com suas palavras o que cada chamada abaixo faz e por que ela
é necessária:

| Chamada                                  | O que ela faz?                                                                                                  |
|------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `socket.socket(AF_INET, SOCK_STREAM)`    | Cria um socket IPv4 (`AF_INET`) e TCP (`SOCK_STREAM`), o ponto de comunicação do servidor.                      |
| `servidor_socket.bind((HOST, PORTA))`    | Associa o socket ao endereço e à porta 5000, para que os clientes saibam onde encontrá-lo.                      |
| `servidor_socket.listen(5)`              | Coloca o socket em modo de escuta, com fila de até cerca de 5 conexões pendentes.                               |
| `servidor_socket.accept()`               | Bloqueia até haver uma conexão e retorna um novo socket (`conn`), exclusivo daquele cliente, e o endereço dele. |
| `conn.recv(TAMANHO_BUFFER)`              | Bloqueia até chegarem dados e lê até 1024 bytes. Retorna vazio se o cliente encerrou a conexão.                 |
| `conn.sendall(resposta.encode("utf-8"))` | Converte a resposta em bytes e envia tudo. `send()` poderia enviar só uma parte.                                |
| `conn.close()`                           | Encerra a conexão com esse cliente e libera o socket; o socket de escuta continua aberto.                       |

### 1.2 Cliente (`cliente.py`)

| Chamada                                                   | O que ela faz?                                                                                    |
|-----------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| `socket.socket(AF_INET, SOCK_STREAM)`                     | Cria o socket IPv4/TCP do cliente. Não precisa de `bind`: o sistema escolhe uma porta livre.      |
| `cliente_socket.connect((HOST_SERVIDOR, PORTA_SERVIDOR))` | Resolve o nome e estabelece a conexão TCP (handshake). Falha se o servidor não estiver escutando. |
| `cliente_socket.sendall(mensagem.encode("utf-8"))`        | Converte a mensagem em bytes UTF-8 e envia todos ao servidor.                                     |
| `cliente_socket.recv(TAMANHO_BUFFER)`                     | Bloqueia até receber a resposta e lê até 1024 bytes.                                              |

### 1.3 Rede e contêineres

Por que o cliente usa o hostname `"servidor"` em vez de `"localhost"`
para se conectar? O que aconteceria se usasse `"localhost"`?

> Cada contêiner tem sua própria interface de rede, e o DNS do Docker resolve `servidor` para o IP do contêiner do servidor. `localhost` seria o próprio contêiner do cliente, onde nada escuta na porta 5000, causando `ConnectionRefusedError`.

---

## Nível 2 — Modificar o servidor

Descreva as mudanças que você fez no `servidor.py` para:

1. Devolver a mensagem **em maiúsculas**:

   > `resposta = mensagem` virou `resposta = mensagem.upper()`, para converter o texto antes de enviá-lo.

2. Exibir no log o **contador de mensagens recebidas** acumulado:

   > Adicionado `total_mensagens += 1` a cada mensagem recebida e incluído `#{total_mensagens}` no log, pois a variável existia mas não era usada.

Após a modificação, você precisou rodar `docker compose up --build`.
Por que o `--build` é necessário aqui?

> O código é copiado para a imagem pelo `COPY` do Dockerfile. Sem `--build`, o Compose reutilizaria a imagem antiga, sem a alteração.

---

## Observações livres

Anote aqui qualquer comportamento inesperado que observaram, erros que
apareceram e como resolveram, ou dúvidas que ainda têm:

> - Sem `depends_on`, o cliente funcionou nas 3 execuções testadas, pois o servidor iniciou antes. Isso não é garantido: `depends_on` controla a ordem de início, mas não espera o servidor estar escutando.
> - Com o servidor parado, o cliente falhou com `socket.gaierror: Temporary failure in name resolution`, pois o nome `servidor` deixou de ser resolvido.
> - Com um segundo cliente, o contador foi para `#2`. Ele fica em memória e zera ao reiniciar o servidor.

---

## Dúvida para a próxima aula

Escreva **uma pergunta** que surgiu durante a atividade e que vocês
gostariam de ver respondida na aula seguinte:

>
