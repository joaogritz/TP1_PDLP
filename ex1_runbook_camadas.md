# Runbook por camadas: Internet no Linux

## 1. Enlace

Nesta etapa verificamos se as interfaces de rede existem e se estão ativas.

### Comando 1

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

**Conclusão:** A interface Wi-Fi `wlp0s20f3` está com estado `UP` e `LOWER_UP`, indicando que está habilitada e possui enlace de rede. A interface Ethernet `enp3s0` está habilitada, mas apresenta `NO-CARRIER` e estado `DOWN`, indicando ausência de enlace.

### Comando 2

```bash
ip -s link
```

Saída:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    RX:  bytes packets errors dropped  missed   mcast
        878154    4289      0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
        878154    4289      0       0       0       0
2: enp3s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 60:c7:27:05:e0:00 brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  missed   mcast
             0       0      0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
             0       0      0       0       0       0
    altname enx60c72705e000
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 12:51:cf:9a:6d:e3 brd ff:ff:ff:ff:ff:ff permaddr 04:e8:b9:b5:95:28
    RX:  bytes packets errors dropped  missed   mcast
    5193365308 3664978      0       3       0       0
    TX:  bytes packets errors dropped carrier collsns
      65348419  390046      0       0       0       0
    altname wlx04e8b9b59528
```

**Conclusão:** A interface `wlp0s20f3` está enviando e recebendo pacotes, sem erros registrados. Há 3 pacotes descartados na recepção e nenhum na transmissão.

---

## 2. Rede (IP)

Nesta etapa verificamos o endereço IP da máquina, as rotas configuradas e a comunicação com outros dispositivos.

### Comando 1

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
       valid_lft 5473sec preferred_lft 5473sec
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259199sec preferred_lft 172799sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**Conclusão:** A interface `wlp0s20f3` possui o endereço IPv4 `192.168.0.12/24`, além de endereços IPv6 global e local de enlace, indicando que existe uma configuração de IP.

### Comando 2

```bash
ip route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** Existe uma rota padrão através do gateway `192.168.0.1` pela interface `wlp0s20f3`. Isso permite que pacotes destinados a redes externas sejam encaminhados pelo roteador.

### Teste adicional

```bash
ping -c 4 8.8.8.8
```

Saída:

```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=7.01 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=7.71 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=5.01 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=7.07 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 5.005/6.697/7.709/1.014 ms
```

**Conclusão:** A resposta do endereço `8.8.8.8` indica que existe conectividade IP com a Internet.

---

## 3. Transporte (TCP/UDP)

Nesta etapa verificamos conexões e portas TCP/UDP.

### Comando 1

```bash
ss -tuln
```

Saída:

```text
Netid  State    Recv-Q   Send-Q                            Local Address:Port                                   Peer Address:Port
udp    UNCONN   0        0                                   224.0.0.251:5353                                        0.0.0.0:*
udp    UNCONN   0        0                                       0.0.0.0:5353                                        0.0.0.0:*
udp    UNCONN   0        0                                       0.0.0.0:5355                                        0.0.0.0:*
udp    UNCONN   0        0                                  192.168.0.12:3702                                        0.0.0.0:*
udp    UNCONN   0        0                               239.255.255.250:3702                                        0.0.0.0:*
udp    UNCONN   0        0                                    127.0.0.54:53                                          0.0.0.0:*
udp    UNCONN   0        0                                 127.0.0.53%lo:53                                          0.0.0.0:*
udp    UNCONN   0        0                                     127.0.0.1:323                                         0.0.0.0:*
udp    UNCONN   0        0                                       0.0.0.0:58279                                       0.0.0.0:*
udp    UNCONN   0        0                                          [::]:5353                                           [::]:*
udp    UNCONN   0        0                                          [::]:5355                                           [::]:*
udp    UNCONN   0        0          [fe80::652d:e96e:d411:ac4]%wlp0s20f3:3702                                           [::]:*
udp    UNCONN   0        0                           [ff02::c]%wlp0s20f3:3702                                           [::]:*
udp    UNCONN   0        0                                             *:53157                                             *:*
udp    UNCONN   0        0                                         [::1]:323                                            [::]:*
udp    UNCONN   0        0          [fe80::652d:e96e:d411:ac4]%wlp0s20f3:546                                            [::]:*
tcp    LISTEN   0        4096                                  127.0.0.1:43969                                       0.0.0.0:*
tcp    LISTEN   0        4096                                    0.0.0.0:5355                                        0.0.0.0:*
tcp    LISTEN   0        10                                      0.0.0.0:27500                                       0.0.0.0:*
tcp    LISTEN   0        4096                                 127.0.0.54:53                                          0.0.0.0:*
tcp    LISTEN   0        4096                                  127.0.0.1:631                                         0.0.0.0:*
tcp    LISTEN   0        4096                              127.0.0.53%lo:53                                          0.0.0.0:*
tcp    LISTEN   0        4096                                       [::]:5355                                           [::]:*
tcp    LISTEN   0        4096                                      [::1]:631                                            [::]:*
```

**Conclusão:** Existem serviços utilizando portas TCP e UDP no sistema. O comando permite identificar quais portas estão aguardando conexões.

### Comando 2

```bash
ss -tn
```

Saída:

```text
State       Recv-Q  Send-Q                                 Local Address:Port                                          Peer Address:Port
ESTAB       0       0                                                 192.168.0.12:55640                                           35.186.238.177:443
ESTAB       0       0                                                 192.168.0.12:39950                                           35.227.232.229:443
ESTAB       0       0                                                 192.168.0.12:51954                                            20.184.175.23:443
ESTAB       0       0                                                 192.168.0.12:47362                                              13.107.5.93:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:48774                                [2800:3f0:4004:80e::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:50404                                [2800:3f0:4004:80e::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:49740                              [2a06:98c1:3101::6812:202f]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:46360                                     [2600:1901:0:47fc::]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:60656                                   [2001:4860:4842:400::]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:53648                                [2800:3f0:4004:815::200e]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:59838                              [2a06:98c1:3101::6812:202f]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:39956                                 [2800:3f0:4003:c00::bc]:5228
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:46494                                  [2800:3f0:4001:c03::5e]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:60780                                [2800:3f0:4004:815::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:51244                                [2800:3f0:4004:80d::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:54496                              [2606:4700:440b::6812:2715]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:54508                              [2606:4700:440b::6812:2715]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:54514                              [2606:4700:440b::6812:2715]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:34034                                [2800:3f0:4004:811::200e]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:45956                                [2800:3f0:4004:80f::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:50334                                [2800:3f0:4004:80f::200a]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:48934                                [2800:3f0:4004:811::200e]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:32822                                     [2600:1901:0:7a0b::]:443
CLOSE-WAIT  76      0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:45410                                      [2a04:4e42:2a::347]:443
CLOSE-WAIT  76      0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:45418                                      [2a04:4e42:2a::347]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:34764                                       [2603:1061:10::16]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:33722                              [2a06:98c1:3101::6812:202f]:443
ESTAB       0       0                    [2804:2f60:4239:1b01:895f:4f1d:fe39:a284]:45406                                [2800:3f0:4004:80c::200a]:443
```

**Conclusão:** Existe uma conexão TCP estabelecida (`ESTAB`) com uma máquina remota na porta `443`, normalmente utilizada por HTTPS. Isso indica que conexões TCP estão funcionando.

---

## 4. Resolução (DNS/hosts)

Nesta etapa verificamos se nomes de domínio estão sendo convertidos corretamente para endereços IP.

### Comando 1

```bash
getent hosts google.com
```

Saída:

```text
2800:3f0:4004:816::200e google.com
```

**Conclusão:** O nome `google.com` foi resolvido para um endereço IP, indicando que a resolução de nomes está funcionando.

### Comando 2

```bash
cat /etc/resolv.conf
```

Saída:

```text
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search .
```

**Conclusão:** O sistema utiliza o resolvedor local do `systemd-resolved` em `127.0.0.53`. O comando `resolvectl status` permite consultar os servidores DNS usados na conexão.

---

## 5. Aplicação

Nesta etapa verificamos se aplicações conseguem utilizar a rede normalmente.

### Comando 1

```bash
curl -I https://www.google.com
```

Saída:

```text
HTTP/2 200
content-type: text/html; charset=ISO-8859-1
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-spIC5SviDXDuOqXrcj5lCQ' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
accept-ch: Sec-CH-Prefers-Color-Scheme
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Thu, 08 Oct 2026 21:06:45 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Thu, 08 Oct 2026 21:06:45 GMT
cache-control: private
set-cookie: __Secure-STRP=ABNhA6iujBu9HhvulHS1RdZojCafXEuHqXIPnphFwSTxOH5-KCItIMrDf0ezrKaJWi8fDh-PTaSyLei0zSHXRqvaDsALl--90HP5; expires=Thu, 08-Oct-2026 21:11:45 GMT; path=/; domain=.google.com; Secure; SameSite=strict
set-cookie: AEC=Aaa9EJpc5K2CnW8e602Y0lsSF1azYPgPqpa-utGXeqh66Y4clWxvvLoD9g; expires=Tue, 06-Apr-2027 21:06:45 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=Cu0BCAESnwEBOxGDSCISv5WCCl8ipaIMFgxuOGUh6p3vsuOxo4ByvD9NIA-1QXQrq0FEd5ajh2wtLjQe988vQ0VY6TyAytl1ENUoCEkS7Vb4suE0bfkgjqUzdkV7-k3ttFij0tWKfNcq4Ia9qtytRpjDcaxwv5oDfxIiFmW_K53egGDrfG5sn_MppUiUBk1uopnp8Rp2J3t_Ng4XuqPhWIV9cUnPP3coATJFAdKsB89ql5l42MeAqwrp9dc2CpimJPfLC_jOrZkUTko3qzUOlgwcpiPcnTOdLXaJwSx30nGKCJjXFNR06ZWyRMdZNGMJ; expires=Fri, 09-Apr-2027 21:06:45 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
```

**Conclusão:** O servidor respondeu com código HTTP `200`, indicando que a aplicação conseguiu estabelecer comunicação HTTP/HTTPS com a Internet.

### Comando 2

```bash
wget --spider https://www.google.com
```

Saída:

```text
[0] Checking 'https://www.google.com' ...
HTTP response 200  [https://www.google.com]
(null) 100% [=============================>]    1.39K    --.-KB/s
                          [Headers: 1 (0 redirects & 0 er]
```

**Conclusão:** O `wget` conseguiu acessar o endereço informado, indicando que a comunicação na camada de aplicação está funcionando.

---

# Hipótese e verificação

## Hipótese 1 – Falha de camada baixa

**Hipótese:** A interface de rede está desativada ou sem conexão.

Para verificar essa hipótese, podemos utilizar:

```bash
ip link
ip addr
```

Se `ip link` apresentar a interface como `DOWN`, existe um problema relacionado à interface ou ao enlace. Se ela estiver `UP` e possuir um endereço IP válido em `ip addr`, essa hipótese pode ser refutada.

Nas saídas deste runbook, a interface Wi-Fi `wlp0s20f3` está `UP`, possui enlace e endereço IP. Portanto, não há evidência de falha de enlace na conexão Wi-Fi utilizada. A interface Ethernet `enp3s0` está sem enlace.

## Hipótese 2 – Falha de camada alta

**Hipótese:** Existe conexão com a Internet, mas o DNS não está funcionando.

Podemos comparar:

```bash
ping -c 4 8.8.8.8
getent hosts google.com
```

Se o `ping 8.8.8.8` funcionar, existe conectividade IP com a Internet. Porém, se `getent hosts google.com` não retornar um endereço, existe uma possível falha na resolução DNS.

Nas saídas deste runbook, tanto a comunicação com `8.8.8.8` quanto a resolução de `google.com` funcionaram. Portanto, a hipótese de falha de DNS é refutada.

## Conclusão

A análise por camadas facilita a identificação de problemas de rede. Primeiro é possível verificar a interface física/lógica, depois a configuração IP e as rotas, as conexões TCP/UDP, a resolução de nomes e, por último, o funcionamento das aplicações.

Dessa forma, caso uma aplicação não consiga acessar a Internet, é possível descobrir em qual camada o problema está ocorrendo sem realizar testes aleatórios.
