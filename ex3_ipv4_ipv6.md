# IPv4 vs IPv6: inventário e prontidão operacional

## 1. Endereços IPv4 e IPv6

### IPv4

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
       valid_lft 4680sec preferred_lft 4680sec
```

**Conclusão:** O IPv4 principal é `192.168.0.12/24`.

### IPv6

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
       valid_lft 259198sec preferred_lft 172798sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**Conclusão:** A interface `wlp0s20f3` possui o IPv6 link-local `fe80::652d:e96e:d411:ac4/64` e o IPv6 global `2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64`.

Foi identificado IPv6 global no prefixo `2804:2f60:4239:1b01::/64`. Não foi identificado endereço ULA nas saídas enviadas.

---

## 2. Rotas IPv4 e IPv6

### IPv4

```bash
ip -4 route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** Existe uma rota default IPv4 através do gateway `192.168.0.1`, pela interface `wlp0s20f3`.

### IPv6

```bash
ip -6 route
```

Saída:

```text
2804:2f60:4239:1b01::/64 dev wlp0s20f3 proto ra metric 600 pref medium
fe80::/64 dev wlp0s20f3 proto kernel metric 1024 pref medium
default via fe80::32de:4bff:fe08:f1f6 dev wlp0s20f3 proto ra metric 600 pref medium
```

**Conclusão:** Existe uma rota default IPv6 através do gateway link-local `fe80::32de:4bff:fe08:f1f6`, pela interface `wlp0s20f3`, aprendida por anúncio de roteador (`proto ra`).

---

## 3. Respostas

### Qual é o IPv4 principal e seu prefixo?

O IPv4 principal é:

```text
192.168.0.12/24
```

### Existe IPv6 fe80::/10? Em qual interface?

Sim. O endereço `fe80::652d:e96e:d411:ac4/64`, pertencente à faixa link-local `fe80::/10`, está na interface `wlp0s20f3`.

### Existe IPv6 global/ULA? Qual prefixo?

Foi identificado IPv6 global no prefixo `2804:2f60:4239:1b01::/64`. Não foi identificado endereço ULA nas saídas enviadas.

### Existe rota default IPv4? E IPv6?

Existe rota default IPv4:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
```

Existe rota default IPv6:

```text
default via fe80::32de:4bff:fe08:f1f6 dev wlp0s20f3 proto ra metric 600 pref medium
```

---

## 4. Resolução de domínio

Foi utilizado o domínio `google.com`, aproveitando a consulta real enviada durante o exercício 2.

```bash
getent hosts google.com
```

Saída:

```text
2800:3f0:4004:80c::200e google.com
```

**Conclusão:** O nome `google.com` foi resolvido para um endereço IPv6. Essa consulta confirma a resolução de nomes; isoladamente, não testa a conectividade com esse endereço.

---

# Conclusão

## Meu host está operacionalmente pronto para IPv6?

**A configuração indica prontidão para IPv6**, considerando as evidências enviadas:

1. A interface `wlp0s20f3` possui o endereço link-local `fe80::652d:e96e:d411:ac4/64`.
2. Existe o endereço IPv6 global `2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64`, no prefixo `2804:2f60:4239:1b01::/64`.
3. Existe uma rota default IPv6 via `fe80::32de:4bff:fe08:f1f6` na interface `wlp0s20f3`.

O host possui endereço e roteamento configurados para comunicação IPv6 além da rede local. Para confirmar a conectividade externa em funcionamento, ainda é necessário um teste específico, como `ping -6 -c 4 2606:4700:4700::1111`.
