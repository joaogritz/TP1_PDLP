# OSI aplicado ao Linux

O modelo OSI pode ser utilizado como referência para organizar o diagnóstico de problemas de rede. No Linux, diferentes comandos permitem observar informações relacionadas a cada camada.

## Tabela OSI → Linux

| Camada OSI | Conceitos-chave | Comandos Linux | Evidência esperada |
|---|---|---|---|
| 2 - Enlace | Interface de rede, MAC, estado do enlace | `ip link` | Interface em estado `UP`, endereço MAC e informações do enlace |
| 3 - Rede | Endereço IP, roteamento, ICMP | `ip addr`, `ip route`, `ping` | IP configurado, rota padrão e resposta ICMP |
| 4 - Transporte | TCP, UDP, portas e conexões | `ss -tuln`, `ss -tn` | Portas abertas e conexões TCP/UDP |
| 5 - Sessão | Estabelecimento e manutenção de sessões | `ss -tn` | Conexões estabelecidas entre máquina local e remota |
| 6 - Apresentação | Criptografia, TLS, formato dos dados | `openssl s_client -connect google.com:443` | Certificado TLS e informações sobre a conexão criptografada |
| 7 - Aplicação | HTTP, DNS e serviços utilizados pelo usuário | `curl`, `getent hosts` | Resposta HTTP e resolução de nomes |

---

# Evidências práticas

Para demonstrar o funcionamento do modelo OSI no Linux, foram escolhidas as camadas 2, 3 e 7.

## Camada 2 - Enlace

### Comando

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

### Evidência

A interface Wi-Fi `wlp0s20f3` aparece com os estados `UP` e `LOWER_UP` e endereço MAC `12:51:cf:9a:6d:e3`, indicando enlace ativo. A interface Ethernet `enp3s0` apresenta `NO-CARRIER` e estado `DOWN`, indicando ausência de enlace.

### Justificativa técnica

O comando `ip link` mostra informações relacionadas às interfaces de rede do sistema.  
Ele permite verificar o estado do enlace e o endereço MAC da interface.  
Essas informações estão associadas à camada 2 do modelo OSI, responsável pela comunicação entre dispositivos dentro de uma rede local.  
Por isso, esse comando é útil para identificar problemas relacionados à interface ou ao enlace.

---

## Camada 3 - Rede

### Comandos

```bash
ip addr
ip route
ping -c 4 8.8.8.8
```

Saída do `ip addr`:

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
       valid_lft 4935sec preferred_lft 4935sec
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259198sec preferred_lft 172798sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Saída do `ip route`:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

Saída do `ping`:

```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=7.64 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=8.28 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=8.41 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=5.38 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 5.384/7.427/8.405/1.214 ms
```

### Evidência

A interface `wlp0s20f3` possui IPv4 `192.168.0.12/24` e endereços IPv6. A rota padrão utiliza o gateway `192.168.0.1`. O teste com `8.8.8.8` recebeu as quatro respostas, com 0% de perda e latência média de 7,427 ms.

### Justificativa técnica

A camada 3 do modelo OSI é responsável pelo endereçamento e roteamento entre redes.  
O `ip addr` permite verificar o endereço IP configurado na máquina, enquanto `ip route` mostra as rotas usadas para encaminhar os pacotes.  
O comando `ping` utiliza ICMP para verificar se existe comunicação entre dois endereços IP.  
Esses comandos permitem observar diretamente o funcionamento da camada de rede.

---

## Camada 7 - Aplicação

### Comandos

```bash
curl -I https://www.google.com
getent hosts google.com
```

Saída do `curl`:

```text
HTTP/2 200
content-type: text/html; charset=ISO-8859-1
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-GMSFCh0GiqOaqRjydWq5_A' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
accept-ch: Sec-CH-Prefers-Color-Scheme
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Thu, 08 Oct 2026 21:15:05 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Thu, 08 Oct 2026 21:15:05 GMT
cache-control: private
set-cookie: __Secure-STRP=ABNhA6gW7wyBYVmwCIFX1BvZOKCm_73eZc4izYX7fOJxn-Zf2Ac0Q9kbBT4bbCBdXWj-IAIN9xhXm2ij78-X-7FtCMMblHA_PA; expires=Thu, 08-Oct-2026 21:20:05 GMT; path=/; domain=.google.com; Secure; SameSite=strict
set-cookie: AEC=Aaa9EJrBorvua0iGzuRQN_bDrpIxJBH-6mY0HRLs8MvWIsjH856t0McOQw; expires=Tue, 06-Apr-2027 21:15:05 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=Cu4BCAESoAEBOxGDSCVp1bQDudZFg5ExeE9PSm8-EYuCm_l0ThD6Oig5TbjLFYUo7HEHZcqsBml-q0xF5mZUfGBE9RDBJMwbTDgEsWAdl6pKxEUS3Ya1VZhL63yey62VikmBBTnyjGzaJMNrgoYrK3ZpPijoUcyiyWgiwuscwGgxQUk1Q4D6uePeg6cKa4qh0UW6THYtMMwPu-J7FI5xz0i_5oG9ZJj9KAEyRQHSrAfPzMeG-If3TE-mMRUxp1yvTqUxjhfpXQyVWdzvxY8C_s6pHM5B5U9z7nnafCdfNKbjJiIzg7B2TlR7zy0iAht7BA; expires=Fri, 09-Apr-2027 21:15:05 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
```

Saída do `getent`:

```text
2800:3f0:4004:80c::200e google.com
```

### Evidência

O sistema resolveu o nome `google.com` para o IPv6 `2800:3f0:4004:80c::200e`. A requisição HTTPS a `www.google.com` retornou `HTTP/2 200`, confirmando o sucesso da comunicação com o servidor.

### Justificativa técnica

A camada 7 representa os protocolos e serviços utilizados diretamente pelas aplicações.  
O comando `curl` permite realizar requisições HTTP ou HTTPS e verificar a resposta fornecida pelo servidor.  
Já `getent hosts` permite verificar a resolução de nomes utilizada pelas aplicações do sistema.  
Assim, esses comandos ajudam a identificar se serviços da camada de aplicação estão funcionando corretamente.

---

# Ambiguidade e limites

Apesar de ser útil para organizar o diagnóstico, o modelo OSI não explica perfeitamente todos os problemas encontrados em redes modernas.

## 1. Protocolos podem envolver mais de uma camada

Alguns protocolos não se encaixam perfeitamente em apenas uma camada. HTTPS, por exemplo, utiliza HTTP na camada de aplicação, TLS para criptografia e TCP na camada de transporte.

Por isso, um problema de acesso HTTPS pode estar relacionado a diferentes pontos da comunicação, e não somente à camada 7.

## 2. O Linux nem sempre separa as camadas de forma clara

Alguns comandos apresentam informações de várias camadas ao mesmo tempo.

Por exemplo:

```bash
ss -tn
```

O comando mostra conexões TCP da camada de transporte, mas essas conexões também podem representar sessões mantidas por aplicações.

Portanto, nem sempre existe uma relação exata de um comando para uma única camada OSI.

## 3. Problemas da aplicação podem ocorrer mesmo com a rede funcionando

Uma aplicação pode falhar mesmo quando todas as camadas de rede estão funcionando corretamente.

Por exemplo, um servidor web pode estar acessível pela rede, mas retornar:

```text
HTTP/1.1 500 Internal Server Error
```

Nesse caso, IP, roteamento e TCP podem estar funcionando normalmente. O problema está no funcionamento interno da aplicação ou do servidor.

Assim, o modelo OSI ajuda a localizar problemas de comunicação, mas nem sempre consegue explicar falhas internas do software.

# Conclusão

O modelo OSI é útil como forma de organizar o diagnóstico de redes no Linux. Comandos como `ip link`, `ip addr`, `ip route`, `ping`, `ss`, `openssl`, `curl` e `getent` permitem observar diferentes partes da comunicação.

Apesar disso, as redes atuais nem sempre seguem uma divisão rígida entre as camadas. Por isso, o OSI deve ser utilizado principalmente como uma referência para organizar a investigação de problemas.