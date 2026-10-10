# DHCP

## 1. Verificação da interface

Primeiro, foi verificada a configuração de rede:

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
       valid_lft 5968sec preferred_lft 5968sec
    inet6 2804:2f60:4239:1b01:895f:4f1d:fe39:a284/64 scope global dynamic noprefixroute
       valid_lft 259196sec preferred_lft 172796sec
    inet6 fe80::652d:e96e:d411:ac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**Conclusão:** A palavra `dynamic` indica que o endereço IPv4 foi obtido dinamicamente, normalmente através de DHCP.

Também é possível verificar a rota:

```bash
ip route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** A presença de `proto dhcp` indica que a rota default foi configurada através de DHCP.

---

## 2. Evidências do lease DHCP

Em sistemas que utilizam NetworkManager, podemos verificar informações da conexão com:

```bash
nmcli device show wlp0s20f3
```

Saída:

```text
GENERAL.DEVICE:                         wlp0s20f3
GENERAL.TYPE:                           wifi
GENERAL.HWADDR:                         12:51:CF:9A:6D:E3
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     JOAO PEDRO 02
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveC>
IP4.ADDRESS[1]:                         192.168.0.12/24
IP4.GATEWAY:                            192.168.0.1
IP4.ROUTE[1]:                           dst = 192.168.0.0/24, nh = 0.0.0.0, mt >
IP4.ROUTE[2]:                           dst = 0.0.0.0/0, nh = 192.168.0.1, mt =>
IP4.DNS[1]:                             192.168.0.1
IP6.ADDRESS[1]:                         2804:2f60:4239:1b01:895f:4f1d:fe39:a284>
IP6.ADDRESS[2]:                         fe80::652d:e96e:d411:ac4/64
IP6.GATEWAY:                            fe80::32de:4bff:fe08:f1f6
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 1024
IP6.ROUTE[2]:                           dst = 2804:2f60:4239:1b01::/64, nh = ::>
IP6.ROUTE[3]:                           dst = ::/0, nh = fe80::32de:4bff:fe08:f>
IP6.DNS[1]:                             fe80::32de:4bff:fe08:f1f6
```

Foi consultado o diretório de estado do NetworkManager:

```bash
ls /var/lib/NetworkManager/
```

Saída:

```text
ls: cannot open directory '/var/lib/NetworkManager/': Permission denied
```

A primeira tentativa não tinha permissão para listar o diretório. A consulta foi repetida com privilégios:

```bash
sudo ls /var/lib/NetworkManager/
```

Saída:

```text
internal-b2bb2ab0-28fe-418d-926e-ca3c641bb53a-wlp0s20f3.lease  seen-bssids
NetworkManager-intern.conf                                      timestamps
secret_key
```

**Conclusão:** O NetworkManager apresenta a interface Wi-Fi conectada e informa endereço, gateway e DNS. A listagem confirma a presença de um arquivo `.lease` associado a `wlp0s20f3`, e seu conteúdo foi consultado posteriormente, conforme a seção 3.

As linhas do `nmcli` terminadas em `>` foram truncadas na visualização enviada e foram preservadas como recebidas. Para obter uma nova saída completa, pode-se utilizar `nmcli --pager no --mode multiline device show wlp0s20f3`.

---

# 3. Parâmetros recebidos

Com base nas saídas:

| Parâmetro | Valor |
|---|---|
| IP | `192.168.0.12/24` |
| Gateway | `192.168.0.1` |
| DNS | `192.168.0.1` |
| Validade IPv4 restante na coleta | `5968sec`, conforme `valid_lft` |
| Duração total do lease | Não informada pelo arquivo consultado |
| Renovação do lease | Não demonstrada nas saídas enviadas |

O valor `valid_lft` representa a validade restante do endereço na coleta; ele não informa a duração original completa do lease nem comprova uma renovação. Detalhes podem ser consultados no arquivo de lease ou nos logs do gerenciador de rede, quando disponíveis.

### Complementação dos dados do lease

O arquivo foi consultado no terminal do host:

```bash
sudo cat /var/lib/NetworkManager/internal-b2bb2ab0-28fe-418d-926e-ca3c641bb53a-wlp0s20f3.lease
```

Saída:

```text
# This is private data. Do not parse.
ADDRESS=192.168.0.12
```

**Conclusão:** O conteúdo confirma o endereço `192.168.0.12` no arquivo associado à interface. O arquivo não informa duração total, instante de expiração ou registros de renovação; portanto, esses dados não podem ser extraídos dessa saída. A consulta foi registrada como evidência, sem depender de análise automatizada desse formato privado.

Consulta opcional dos registros DHCP, caso seja necessário investigar a renovação:

```bash
journalctl -u NetworkManager | grep -i dhcp
```

---

# 4. Cenários de falha

Os cenários abaixo são hipóteses didáticas; não foram observados ou provocados nas saídas enviadas.

## DHCP entrega DNS vazio ou incorreto

Nesse caso, o host pode receber IP e gateway normalmente, mas não conseguir resolver nomes de domínio.

Um acesso utilizando IP pode funcionar:

```bash
ping 8.8.8.8
```

Enquanto um acesso utilizando nome pode falhar:

```bash
ping google.com
```

**Sintoma esperado:** existe conectividade IP, mas nomes de domínio não são resolvidos corretamente.

---

## DHCP não entrega gateway

O host pode receber um endereço IP válido e conseguir acessar dispositivos da própria rede local.

Porém, sem uma rota default, não terá um caminho para outras redes ou para a Internet.

O problema pode ser identificado com:

```bash
ip route
```

**Sintoma esperado:** comunicação dentro da LAN funciona, mas destinos externos ficam inacessíveis.

---

## Lease expira e não renova

O endereço fornecido pelo DHCP possui um período de validade.

Se o lease expirar e o host não conseguir renová-lo, ele pode perder a configuração de rede ou deixar de possuir uma configuração válida para aquele ambiente.

**Sintoma esperado:** a conectividade pode funcionar inicialmente e depois ser perdida quando o lease expirar.

---

# 5. Ações de contingência

Caso o DHCP fique indisponível, duas possíveis ações de contingência são:

1. **Configurar um endereço IP estático temporariamente**, utilizando um IP, máscara e gateway compatíveis com a rede.

2. **Configurar manualmente servidores DNS e rota default**, quando houver um endereço IP válido e o problema estiver nesses parâmetros. Essa ação isolada não resolve a perda ou expiração do endereço IP.

Essas configurações devem ser feitas com cuidado para evitar conflitos de endereço IP com outros dispositivos.

# Conclusão

As evidências mostram IPv4 dinâmico `192.168.0.12/24` na interface `wlp0s20f3`, rota default com `proto dhcp`, gateway e DNS IPv4 `192.168.0.1` e um arquivo de lease associado à interface. Esses dados sustentam o uso de DHCP na configuração IPv4. A renovação do lease e seu tempo total não foram demonstrados. Os parâmetros IPv6 também aparecem no NetworkManager, mas essa saída não determina, por si só, se foram obtidos por DHCPv6 ou por outros mecanismos.
