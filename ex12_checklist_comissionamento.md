# Checklist de comissionamento de conectividade

Este checklist tem como objetivo validar rapidamente se um host Linux está pronto para operar na rede.

| Item | Objetivo | Comando(s) | Evidência esperada | Critério |
|---|---|---|---|---|
| 1 | Validar interface ativa | `ip link` | Interface principal com `UP` | Passou se estiver `UP` |
| 2 | Validar IPv4 | `ip -4 addr` | Endereço IPv4 configurado | Passou se houver IPv4 válido |
| 3 | Validar IPv6 | `ip -6 addr` | Endereço IPv6 configurado | Passou se houver IPv6 esperado |
| 4 | Validar rota default | `ip route` | Linha iniciando com `default via` | Passou se existir rota default |
| 5 | Validar rotas IPv6 | `ip -6 route` | Rotas IPv6 configuradas | Passou se corresponder à configuração esperada |
| 6 | Validar DNS e hosts | `getent hosts google.com` e `cat /etc/hosts` | Nome resolvido e arquivo hosts válido | Passou se a resolução funcionar |
| 7 | Validar hostname | `hostname` | Nome esperado da máquina | Passou se o hostname estiver correto |
| 8 | Validar portas TCP | `ss -ltn` | Portas TCP em estado `LISTEN` | Passou se as escutas corresponderem ao inventário identificado no exercício 9 |
| 9 | Validar DHCP | `ip addr` e `ip route` | `dynamic` e/ou `proto dhcp` | Passou se DHCP estiver configurado quando esperado |
| 10 | Validar forwarding | `sysctl net.ipv4.ip_forward` | Valor `0` ou valor esperado | Passou se corresponder à função do host |

---

# Evidências executadas

As saídas de interfaces, rotas, resolução de domínio, sockets e configuração DHCP foram enviadas pelo usuário. As consultas de hostname e forwarding foram coletadas diretamente na revisão anterior; o conteúdo final de `/etc/hosts` foi enviado pelo usuário após a restauração. Todos os blocos abaixo apresentam evidências reais; os resultados distinguem configuração confirmada de pontos que exigem avaliação.

## 1. Interface ativa

```bash
ip link
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 60:c7:27:05:e0:00 brd ff:ff:ff:ff:ff:ff
    altname enx60c72705e000
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 12:51:cf:9a:6d:e3 brd ff:ff:ff:ff:ff:ff permaddr 04:e8:b9:b5:95:28
    altname wlx04e8b9b59528
```

**Resultado:** Passou. A interface Wi-Fi `wlp0s20f3` apresenta `UP` e `LOWER_UP`. A Ethernet `enp3s0` está sem enlace (`NO-CARRIER`), mas não é a conexão utilizada.

---

## 2. Endereço IPv4

```bash
ip -4 addr
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    altname wlx04e8b9b59528
    inet 192.168.0.12/24 brd 192.168.0.255 scope global dynamic noprefixroute wlp0s20f3
       valid_lft 5470sec preferred_lft 5470sec
```

**Resultado:** Passou. A interface `wlp0s20f3` possui IPv4 dinâmico `192.168.0.12/24`.

---

## 3. Endereço IPv6

```bash
ip -6 addr
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP qlen 1000
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259192sec preferred_lft 172792sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**Resultado:** Passou para endereçamento. A interface possui IPv6 global `2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64` e link-local `fe80::652d:e96e:d411:ac4/64`. Isso não testa conectividade externa.

---

## 4. Rota default

```bash
ip route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Resultado:** Passou. Existe rota default IPv4 via `192.168.0.1` na interface `wlp0s20f3`, com métrica `600`.

---

## 5. Rotas IPv6

```bash
ip -6 route
```

Saída:

```text
2804:2f60:4239:1b01::/64 dev wlp0s20f3 proto ra metric 600 pref medium
fe80::/64 dev wlp0s20f3 proto kernel metric 1024 pref medium
default via fe80::32de:4bff:fe08:f1f6 dev wlp0s20f3 proto ra metric 600 pref medium
```

**Resultado:** Passou para roteamento configurado. Há rota para o prefixo global, para o enlace local e rota default via `fe80::32de:4bff:fe08:f1f6`, aprendida por anúncio de roteador (`proto ra`).

---

## 6. DNS e hosts

```bash
getent hosts google.com
```

Saída:

```text
2800:3f0:4004:816::200e google.com
```

Também foi verificado:

```bash
cat /etc/hosts
```

Saída:

```text
# Loopback entries; do not change.
# For historical reasons, localhost precedes localhost.localdomain:
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# See hosts(5) for proper format and other examples:
# 192.168.1.10 foo.example.org foo
# 192.168.1.13 bar.example.org bar
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1	kubernetes.docker.internal
# End of section
```

**Resultado:** Passou. A resolução de `google.com` retornou o IPv6 `2800:3f0:4004:816::200e`. A coleta final de `/etc/hosts` confirma a restauração da linha original de `::1` e a remoção da associação de teste `dr4-joao-linux`.

---

## 7. Hostname

```bash
hostname
```

Saída:

```text
fedora
```

**Resultado:** Passou. O hostname coletado é `fedora`, conforme o nome restaurado no exercício 8.

---

## 8. Portas TCP

```bash
ss -ltn
```

Saída:

```text
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port
LISTEN    0         4096             127.0.0.1:43969            0.0.0.0:*
LISTEN    0         128              127.0.0.1:40319            0.0.0.0:*
LISTEN    0         4096               0.0.0.0:5355             0.0.0.0:*
LISTEN    0         10                 0.0.0.0:27500            0.0.0.0:*
LISTEN    0         4096            127.0.0.54:53               0.0.0.0:*
LISTEN    0         4096             127.0.0.1:631              0.0.0.0:*
LISTEN    0         4096         127.0.0.53%lo:53               0.0.0.0:*
LISTEN    0         511              127.0.0.1:41989            0.0.0.0:*
LISTEN    0         4096                  [::]:5355                [::]:*
LISTEN    0         4096                 [::1]:631                 [::]:*
```

**Resultado:** Passou na comparação com o inventário do exercício 9. Todas as portas e endereços desta coleta correspondem às escutas identificadas anteriormente:

| Porta TCP | Serviço identificado no exercício 9 | Endereço de escuta |
|---|---|---|
| 43969 | Docker Desktop | `127.0.0.1` |
| 40319 | Codex | `127.0.0.1` |
| 5355 | systemd-resolved | `0.0.0.0` e `[::]` |
| 27500 | passim | `0.0.0.0` |
| 53 | systemd-resolved | `127.0.0.54` e `127.0.0.53%lo` |
| 631 | CUPS | `127.0.0.1` e `[::1]` |
| 41989 | VS Code | `127.0.0.1` |

Não foram observadas escutas adicionais em relação a esse inventário, e a porta de teste `55000` não aparece. A associação dos serviços vem da coleta do exercício 9; `ss -ltn` não identifica processos. Esse resultado valida a correspondência com o inventário, sem representar uma auditoria de firewall ou autorização de acesso.


---

## 9. DHCP

```bash
ip addr
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp3s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 60:c7:27:05:e0:00 brd ff:ff:ff:ff:ff:ff
    altname enx60c72705e000
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 12:51:cf:9a:6d:e3 brd ff:ff:ff:ff:ff:ff permaddr 04:e8:b9:b5:95:28
    altname wlx04e8b9b59528
    inet 192.168.0.12/24 brd 192.168.0.255 scope global dynamic noprefixroute wlp0s20f3
       valid_lft 5429sec preferred_lft 5429sec
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259198sec preferred_lft 172798sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Também:

```bash
ip route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Resultado:** Passou para configuração DHCP IPv4. O endereço `192.168.0.12/24` apresenta `dynamic` e a rota default apresenta `proto dhcp`. A renovação do lease não foi testada.

---

## 10. Forwarding IPv4

```bash
sysctl net.ipv4.ip_forward
```

Saída:

```text
net.ipv4.ip_forward = 0
```

**Resultado:** Passou para um host comum. O parâmetro de encaminhamento IPv4 está desabilitado (`0`).

---

# Decisões técnicas

## Rotas

Em caso de falha de roteamento, primeiro verificaria a tabela atual com `ip route`.  
Uma rota só deve ser alterada após identificar qual destino está sendo encaminhado incorretamente.  
Sempre que possível, utilizaria uma rota específica, como `/32`, em vez de alterar uma rede inteira ou a rota default.  
Isso reduz o impacto da mudança e facilita a reversão caso a alteração não resolva o problema.

## Resolução

O `/etc/hosts` é útil para testes, overrides locais ou correções temporárias.  
Ele permite associar diretamente um nome a um endereço IP sem depender de DNS.  
Porém, não é adequado como solução permanente em ambientes com várias máquinas.  
Nesses casos, o DNS é preferível porque centraliza a configuração e facilita mudanças de endereço.  
O `/etc/hosts` deve ser utilizado apenas quando houver uma necessidade específica e controlada.

# Conclusão

Os dez itens passaram nos critérios de configuração e comparação descritos neste checklist, com base nas evidências reais coletadas. O host apresenta enlace Wi-Fi ativo, IPv4 e IPv6 configurados, rotas default IPv4 e IPv6, resolução de nomes funcional, hostname `fedora` e forwarding IPv4 desabilitado. A configuração IPv4 indica uso de DHCP, o `/etc/hosts` foi restaurado e as escutas TCP correspondem ao inventário do exercício 9, sem a porta temporária `55000`.

A aprovação se limita aos critérios deste exercício. As coletas não demonstram renovação DHCP ou conectividade IPv6 externa e não constituem uma auditoria das regras de firewall ou da autorização dos serviços.
