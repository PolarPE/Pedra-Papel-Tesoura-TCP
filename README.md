Jogo Pedra, Papel e Tesoura (3 jogadores) - Java Sockets

Integrantes:
- NOME 1
- NOME 2
- NOME 3

Descrição do jogo:
- Jogo de Pedra, Papel e Tesoura para 3 jogadores usando sockets TCP.
- O servidor controla toda a lógica do jogo (ordem de jogadas, validação, resultados, rematch) e mantém as conexões abertas entre rodadas.

Como compilar:
```bash
cd /home/italo/fazendo
javac servidor.java Client1.java Client2.java Client3.java
```

Como executar:
1) Inicie o servidor:
```bash
java servidor
```
2) Em três terminais separados, conecte 3 clientes:
```bash
java Client1
java Client2
java Client3
```

Comportamento interativo dos clientes:
- Ao executar um cliente em **terminal interativo** (sem redirecionamento de stdin), o cliente perguntará:

  Conectar ao servidor <host>:<port>? (S/N):

  Responda `S` ou `SIM` para conectar ou `N`/`NAO` para cancelar.
- Quando a execução for **não interativa** (por exemplo: `java Client1 < c1.txt`), o cliente **conecta automaticamente** — isso mantém compatibilidade com testes scriptados.

Testes automatizados (scriptados):
- Crie três arquivos de entrada (ex.: `c1.txt`, `c2.txt`, `c3.txt`) com sequências de comandos. Exemplo:

`c1.txt`:
```
CONFIRMA
JOGADA R
REMATCH SIM
```

`c2.txt`:
```
CONFIRMA
JOGADA P
REMATCH SIM
```

`c3.txt`:
```
CONFIRMA
JOGADA S
REMATCH SIM
```

- Execute o servidor e os clientes (em background), salvando logs:
```bash
java servidor
java Client1 < c1.txt &> c1.log &
java Client2 < c2.txt &> c2.log &
java Client3 < c3.txt &> c3.log &
```
- Verifique os logs com `tail -f c1.log` (ou `tail -n +1 c1.log`).

Dicas de depuração:
- Se `java servidor` falhar com "Address already in use", finalize o processo antigo com `pkill -f servidor` ou mate o PID.
- Se um cliente encerrar, verifique se recebeu `FIM` do servidor ou se o arquivo de testes terminou (stdin).
Como jogar (fluxo):
- Após os 3 jogadores se conectarem, o servidor pede que cada jogador confirme o início (`PEDIR_CONFIRMACAO`).
- Quando todos confirmarem, a partida inicia.
- Em cada rodada, o servidor aceita jogadas (`JOGADA R|P|S`) e controla a ordem das jogadas (cada jogador será notificado com `SUA_VEZ`, mas pode enviar a jogada a qualquer momento).
- O servidor processa as jogadas na ordem de chegada e garante que cada jogador só tenha uma jogada por rodada.
- Após todas as jogadas, o servidor calcula e informa o resultado a todos (`VENCEU` / `PERDEU` / `EMPATE`) e envia o estado das jogadas (`ESTADO P1:R P2:P P3:S`).
- Após a rodada, o servidor pergunta se desejam jogar novamente (`PERGUNTA_REMATCH`). Se todos aceitarem, inicia nova rodada usando as mesmas conexões.
- Se algum jogador escolher não jogar novamente, a conexão será encerrada e o servidor finaliza.

Protocolo (mensagens principais):
- Servidor -> Cliente:
  - `ASSIGN <id>`: atribui id do jogador
  - `MENSAGEM <texto>`: mensagem informativa
  - `PEDIR_CONFIRMACAO`: pedir confirmação para iniciar a partida
  - `SUA_VEZ`: notifica que é a sua vez (mas ainda aceita jogadas fora de vez)
  - `ESTADO P1:<X> P2:<Y> P3:<Z>`: estado atual das jogadas
  - `VENCEU` / `PERDEU` / `EMPATE`: resultado da rodada
  - `PERGUNTA_REMATCH`: pergunta se deseja jogar novamente
  - `FIM`: encerra a conexão
- Cliente -> Servidor:
  - `CONFIRMA` / `SAIR`: confirmar ou sair antes do início
  - `JOGADA <R|P|S>`: enviar jogada (R=Rock(Pedra), P=Paper(Papel), S=Scissors(Tesoura))
  - `REMATCH SIM` / `REMATCH NAO`: resposta ao rematch

Observações:
- O servidor utiliza uma fila global para processar as mensagens na ordem de chegada, permitindo lidar com jogadas enviadas simultaneamente.
- A ordem de jogadas é mostrada no console a cada jogada e o servidor envia o estado parcial (`ESTADO`) sempre que uma nova jogada é aceita.
- O jogo suporta múltiplas rodadas sem precisar reconectar os clientes.