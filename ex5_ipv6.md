# Configuração IPv6

## 1. Endereços IPv6

Comando utilizado:

```bash
ip -6 addr show
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP qlen 1000
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259196sec preferred_lft 172796sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

### Classificação

| Interface | Endereço IPv6 | Tipo |
|---|---|---|
| lo | `::1/128` | Loopback |
| wlp0s20f3 | `2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64` | Global |
| wlp0s20f3 | `fe80::652d:e96e:d411:ac4/64` | Link-local |

**Conclusão:** A interface `wlp0s20f3` possui um endereço IPv6 global no prefixo `2804:2f60:4239:1b01::/64` e um endereço link-local. Não foi identificado endereço ULA nas saídas enviadas.

---

## 2. Rotas IPv6

Comando utilizado:

```bash
ip -6 route show
```

Saída:

```text
2804:2f60:4239:1b01::/64 dev wlp0s20f3 proto ra metric 600 pref medium
fe80::/64 dev wlp0s20f3 proto kernel metric 1024 pref medium
default via fe80::32de:4bff:fe08:f1f6 dev wlp0s20f3 proto ra metric 600 pref medium
```

**Conclusão:** Existe rota default IPv6 via `fe80::32de:4bff:fe08:f1f6`, pela interface `wlp0s20f3`. As rotas com `proto ra` foram aprendidas por anúncios de roteador.

A rota do prefixo global `2804:2f60:4239:1b01::/64` e a rota link-local `fe80::/64` utilizam a mesma interface.

---

## 3. Comparação entre interfaces

As saídas enviadas mostram endereços IPv6 nas interfaces `lo` e `wlp0s20f3`.

### Interface principal

A interface Wi-Fi `wlp0s20f3` está em estado `UP` e possui:

```text
2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64
fe80::652d:e96e:d411:ac4/64
```

Tipos: global e link-local, respectivamente.

### Interface de loopback

A interface `lo` possui:

```text
::1/128
```

Tipo: loopback, utilizado para comunicação com o próprio host.

A comparação utiliza as duas interfaces com endereços IPv6 presentes na saída: a interface de rede `wlp0s20f3` e a interface lógica de loopback `lo`. A saída não apresenta uma interface de bridge como `docker0`.

### Por que podem ser diferentes?

Cada interface atende a uma finalidade. A interface `wlp0s20f3` conecta o host à rede e possui endereços global e link-local. A interface `lo` permite comunicação interna com o próprio host, utilizando `::1`.

---

# Rota default IPv6

A rota default observada é:

```text
default via fe80::32de:4bff:fe08:f1f6 dev wlp0s20f3 proto ra metric 600 pref medium
```

Ela permite encaminhar tráfego destinado a redes IPv6 externas para o roteador. O gateway utiliza um endereço link-local, o que é compatível com uma rota default IPv6: ele é alcançado diretamente pelo enlace da interface `wlp0s20f3`.

---

# 3 causas prováveis

As causas abaixo são hipóteses gerais para um host que possui somente IPv6 link-local ou não consegue acessar redes IPv6 externas. As saídas atuais mostram endereço global e rota default, portanto não demonstram esse cenário de falha.

## 1. Roteador sem suporte ou configuração IPv6

O roteador pode não fornecer conectividade IPv6 ou não anunciar um prefixo global para os dispositivos. Nas saídas atuais, há um endereço global e rotas aprendidas por anúncios de roteador.

## 2. Ausência de rota default IPv6

Sem uma rota adequada para o destino, o host não consegue encaminhar pacotes para redes externas. Na configuração observada, existe uma rota default IPv6.

## 3. Provedor ou rede sem IPv6 disponível

Uma rede pode oferecer apenas IPv4 ou apresentar falhas no encaminhamento IPv6. A presença de endereço global e rota default no host não comprova, por si só, que o caminho externo funciona.

# Conclusão

O host possui endereço IPv6 global, endereço link-local e rota default na interface `wlp0s20f3`, indicando configuração para comunicação IPv6 além da rede local.

As saídas de endereços e rotas não confirmam a conectividade com a Internet. Para verificar o funcionamento externo, pode-se realizar um teste como `ping -6 -c 4 2606:4700:4700::1111`.
