# Portas TCP

## 1. Portas TCP em escuta

As saídas de sockets abaixo têm o espaçamento entre colunas compactado.

Comando utilizado:

```bash
ss -tulpen | grep tcp
```

Saída:

```text
tcp LISTEN 0 4096 127.0.0.1:43969 0.0.0.0:* uid:1000 ino:210696 sk:1011 cgroup:/user.slice/user-1000.slice/user@1000.service/app.slice/docker-desktop.service <->
tcp LISTEN 0 128 127.0.0.1:40319 0.0.0.0:* users:(("codex",pid=25110,fd=35)) uid:1000 ino:238539 sk:a cgroup:/user.slice/user-1000.slice/user@1000.service/app.slice/ptyxis-spawn-aebf3c50-d470-41f5-9965-414a0aa0910d.scope <->
tcp LISTEN 0 4096 0.0.0.0:5355 0.0.0.0:* uid:193 ino:206256 sk:1012 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 10 0.0.0.0:27500 0.0.0.0:* uid:968 ino:45703 sk:1013 cgroup:/system.slice/passim.service <->
tcp LISTEN 0 4096 127.0.0.54:53 0.0.0.0:* uid:193 ino:206273 sk:1014 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 4096 127.0.0.1:631 0.0.0.0:* ino:40287 sk:1015 cgroup:/system.slice/system-cups.slice/cups.service <->
tcp LISTEN 0 4096 127.0.0.53%lo:53 0.0.0.0:* uid:193 ino:206271 sk:1016 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 511 127.0.0.1:41989 0.0.0.0:* users:(("code",pid=17812,fd=19)) uid:1000 ino:365177 sk:b cgroup:/user.slice/user-1000.slice/user@1000.service/app.slice/app-flatpak-com.visualstudio.code-275678535.scope <->
tcp LISTEN 0 4096 [::]:5355 [::]:* uid:193 ino:206264 sk:1017 cgroup:/system.slice/systemd-resolved.service v6only:1 <->
tcp LISTEN 0 4096 [::1]:631 [::]:* ino:40286 sk:1018 cgroup:/system.slice/system-cups.slice/cups.service v6only:1 <->
```

**Conclusão:** A saída mostra escuta TCP nas portas `43969`, `40319`, `5355`, `27500`, `53`, `631` e `41989`. Não há evidência de escuta na porta `22` nessa coleta.

Os campos `users:` identificam diretamente `codex` (PID `25110`) e `code` (PID `17812`). Os campos `cgroup:` associam outros sockets a serviços, mas não informam seus PIDs. Para obter mais detalhes de processos, pode-se utilizar `sudo ss -tulpen`.

---

## 2. Identificação de serviços e diferença entre UID e porta

Foram executados:

```bash
ss -tulpen | grep ':193 '
```

Saída:

```text
udp UNCONN 0 0 0.0.0.0:5355 0.0.0.0:* uid:193 ino:206255 sk:1003 cgroup:/system.slice/systemd-resolved.service <->
udp UNCONN 0 0 127.0.0.54:53 0.0.0.0:* uid:193 ino:206272 sk:1006 cgroup:/system.slice/systemd-resolved.service <->
udp UNCONN 0 0 127.0.0.53%lo:53 0.0.0.0:* uid:193 ino:206270 sk:1007 cgroup:/system.slice/systemd-resolved.service <->
udp UNCONN 0 0 [::]:5355 [::]:* uid:193 ino:206263 sk:100b cgroup:/system.slice/systemd-resolved.service v6only:1 <->
tcp LISTEN 0 4096 0.0.0.0:5355 0.0.0.0:* uid:193 ino:206256 sk:1012 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 4096 127.0.0.54:53 0.0.0.0:* uid:193 ino:206273 sk:1014 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 4096 127.0.0.53%lo:53 0.0.0.0:* uid:193 ino:206271 sk:1016 cgroup:/system.slice/systemd-resolved.service <->
tcp LISTEN 0 4096 [::]:5355 [::]:* uid:193 ino:206264 sk:1017 cgroup:/system.slice/systemd-resolved.service v6only:1 <->
```

```bash
ss -tulpen | grep ':968 '
```

Saída:

```text
tcp LISTEN 0 10 0.0.0.0:27500 0.0.0.0:* uid:968 ino:45703 sk:1013 cgroup:/system.slice/passim.service <->
```

**Conclusão:** Esses filtros encontraram `uid:193` e `uid:968`. Os números são identificadores de usuários dos sockets, não portas. Os sockets associados a `systemd-resolved` usam as portas `53` e `5355`; o socket associado a `passim` usa a porta `27500`.

Como exemplo de porta comum, a porta `53` aparece em escuta TCP nos endereços locais `127.0.0.53%lo` e `127.0.0.54`, associada ao resolvedor local.

---

## 3. Porta alta escolhida para o serviço de teste

Foi utilizada a porta TCP `55000`, pertencente à faixa de portas altas. Os comandos abaixo foram executados em dois terminais: o primeiro manteve o servidor ativo enquanto o segundo verificou a escuta e tentou iniciar outra instância.

Não foi enviada uma consulta anterior à inicialização. A inicialização bem-sucedida demonstra que o servidor conseguiu vincular o socket nessa porta naquele momento.

---

## 4. Inicialização e verificação do servidor

No terminal 1:

```bash
sudo python3 -m http.server 55000
```

Saída:

```text
Serving HTTP on 0.0.0.0 port 55000 (http://0.0.0.0:55000/) ...
```

Com o servidor ainda em execução, no terminal 2:

```bash
ss -ltnp 'sport = :55000'
```

Saída:

```text
State   Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process
LISTEN  0        5                0.0.0.0:55000          0.0.0.0:*
```

**Conclusão:** A porta `55000` estava em escuta TCP em todos os endereços IPv4 locais. A coluna `Process` não trouxe detalhes, portanto essa saída não informa o PID. A execução registrada no terminal 1 identifica o serviço de teste como o servidor HTTP do Python.

O teste foi executado com `sudo`, conforme o registro real. Uma porta alta como `55000` normalmente pode ser utilizada sem privilégios administrativos; este teste não avaliou essa possibilidade.

---

## 5. Conflito de porta

Com o servidor do terminal 1 ainda ativo, foi executado no terminal 2:

```bash
sudo python3 -m http.server 55000
```

Saída:

```text
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "/usr/lib64/python3.14/http/server.py", line 1432, in <module>
    test(
    ~~~~^
        HandlerClass=handler_class,
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    ...<6 lines>...
        tls_password=tls_key_password,
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "/usr/lib64/python3.14/http/server.py", line 1348, in test
    server = ServerClass(addr, HandlerClass)
  File "/usr/lib64/python3.14/socketserver.py", line 457, in __init__
    self.server_bind()
    ~~~~~~~~~~~~~~~~^^
  File "/usr/lib64/python3.14/http/server.py", line 1419, in server_bind
    return super().server_bind()
           ~~~~~~~~~~~~~~~~~~~^^
  File "/usr/lib64/python3.14/http/server.py", line 148, in server_bind
    socketserver.TCPServer.server_bind(self)
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^
  File "/usr/lib64/python3.14/socketserver.py", line 478, in server_bind
    self.socket.bind(self.server_address)
    ~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^
OSError: [Errno 98] Address already in use
```

**Conclusão:** A segunda instância não conseguiu vincular o socket porque o endereço e a porta já estavam em uso pela primeira instância. Executar com `sudo` não elimina esse conflito.

---

## 6. Encerramento e verificação final

No terminal 1, o servidor foi encerrado com `Ctrl+C`.

Saída:

```text
^C
Keyboard interrupt received, exiting.
```

Depois, no mesmo terminal:

```bash
ss -ltnp 'sport = :55000'
```

Saída:

```text
State   Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process
```

**Conclusão:** O comando retornou apenas o cabeçalho, sem sockets em escuta na porta `55000`. Isso confirma a liberação da porta após o encerramento do servidor.

---

# 0.0.0.0 vs 127.0.0.1

Um serviço em `127.0.0.1` aceita conexões IPv4 locais pela interface de loopback. Um serviço em `0.0.0.0` escuta em todos os endereços IPv4 locais, podendo receber conexões de outros dispositivos se o roteamento e o firewall permitirem.

Na coleta, o servidor Python utilizou `0.0.0.0:55000`. O resolvedor DNS, o CUPS e outros serviços aparecem em endereços de loopback. Em IPv6, `[::1]` representa loopback e `[::]` representa todos os endereços IPv6 locais; as linhas com `v6only:1` atendem somente IPv6.

# Conclusão

As saídas mostram portas TCP em escuta e confirmam o teste com a porta alta `55000`: inicialização do servidor HTTP do Python, verificação da escuta, conflito ao tentar iniciar uma segunda instância no mesmo endereço e porta e liberação após `Ctrl+C`.

Os filtros iniciais também demonstram a diferença entre UID e porta. Os detalhes de processo e PID só podem ser identificados quando aparecem na saída; na consulta da porta `55000`, a coluna `Process` ficou vazia.
