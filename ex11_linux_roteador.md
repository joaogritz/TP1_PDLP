# Linux como roteador

## 1. Estado atual do forwarding IPv4

Comando utilizado:

```bash
sysctl net.ipv4.ip_forward
```

Saída:

```text
net.ipv4.ip_forward = 0
```

**Conclusão:** No início do teste, o encaminhamento IPv4 estava desabilitado. As duas consultas retornaram o valor `0`.

O estado inicial também foi verificado com:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Saída:

```text
0
```

---

## 2. Habilitando temporariamente

Para habilitar o forwarding IPv4:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Saída:

```text
net.ipv4.ip_forward = 1
```

Depois, a linha de resultado foi digitada diretamente no terminal:

```bash
net.ipv4.ip_forward = 1
```

Saída:

```text
bash: net.ipv4.ip_forward: command not found...
```

Essa linha é a saída do `sysctl`, não um comando do shell. A tentativa não alterou o parâmetro. A alteração já havia sido feita pelo comando `sudo sysctl -w net.ipv4.ip_forward=1`.


Verificação:

```bash
sysctl net.ipv4.ip_forward
```

Saída:

```text
net.ipv4.ip_forward = 1
```

**Conclusão:** O encaminhamento IPv4 foi habilitado e a consulta confirmou o valor `1`. A alteração foi realizada em tempo de execução; não foi criada uma configuração persistente em arquivo neste teste.

---

## 3. O que ip_forward=1 permite

Quando `net.ipv4.ip_forward=1`, o Linux pode encaminhar pacotes IPv4 recebidos em uma interface para outra interface.

Isso permite que o host participe do roteamento entre redes diferentes, em vez de processar apenas pacotes destinados a ele próprio.

Porém, somente habilitar essa opção não transforma automaticamente a máquina em um roteador completamente funcional.

---

## 4. O que ainda seria necessário

Em um cenário real, também seria necessário:

1. Ter pelo menos duas interfaces ou caminhos conectados a redes diferentes.
2. Possuir rotas corretas para as redes envolvidas.
3. Configurar regras de firewall para controlar quais pacotes podem ser encaminhados.
4. Em alguns cenários, configurar NAT para permitir que uma rede privada acesse outra rede ou a Internet.

---

## 5. Reversão

Para desabilitar novamente o forwarding:

```bash
sudo sysctl -w net.ipv4.ip_forward=0
```

Saída:

```text
net.ipv4.ip_forward = 0
```

Verificação:

```bash
sysctl net.ipv4.ip_forward
```

Saída:

```text
net.ipv4.ip_forward = 0
```

**Conclusão:** O forwarding IPv4 foi desabilitado e o estado inicial foi restaurado.

---

# Riscos de segurança

Manter o forwarding habilitado sem políticas adequadas pode gerar os seguintes riscos:

1. **Encaminhamento de tráfego não autorizado:** dispositivos podem utilizar o host como caminho para acessar outras redes.

2. **Aumento da superfície de ataque:** redes que deveriam estar isoladas podem passar a ter comunicação entre si.

3. **Contorno de políticas de segurança:** o host pode criar um caminho de rede não planejado, permitindo que tráfego evite controles existentes em outros equipamentos.

# Conclusão

A opção `net.ipv4.ip_forward` controla se o Linux pode encaminhar pacotes IPv4 entre interfaces. Habilitá-la é um dos requisitos para utilizar um host Linux como roteador, mas ainda são necessárias rotas adequadas e políticas de segurança. As saídas confirmam a sequência `0 → 1 → 0`: o forwarding foi habilitado, verificado e desabilitado novamente, restaurando o valor inicial. Não foi apresentado um teste de encaminhamento de pacotes entre redes; o experimento verificou a alteração do parâmetro.