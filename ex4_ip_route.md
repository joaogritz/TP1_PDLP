# Routing table com ip route

## 1. Tabela de rotas

Comando utilizado:

```bash
ip route
```

Saída:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600 
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600 
```

---

## 2. Identificação das rotas

### Rota default

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
```

- Gateway: `192.168.0.1`
- Interface: `wlp0s20f3`
- Métrica: `600`

**Conclusão:** Todo destino que não corresponde a uma rota mais específica será enviado pelo gateway `192.168.0.1`.

### Rota conectada

```text
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

- Prefixo: `192.168.0.0/24`
- Interface: `wlp0s20f3`

**Conclusão:** Destinos dentro da rede `192.168.0.0/24` são acessados diretamente pela interface `wlp0s20f3`.

A tabela enviada contém uma rota default e uma rota conectada. Não há rota para uma rede Docker nessa saída.

---

# 3. Simulação de decisões

Os destinos abaixo são casos de simulação analisados com base na tabela real apresentada. Não representam saídas de comandos executados.

## Destino 1 - Local

Destino:

```text
192.168.0.50
```

A rota que vence é:

```text
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Justificativa:** O endereço `192.168.0.50` pertence à rede `192.168.0.0/24`. Essa rota é mais específica que a rota default, por isso ela é utilizada.

---

## Destino 2 - Público

Destino:

```text
8.8.8.8
```

A rota que vence é:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
```

**Justificativa:** Não existe uma rota específica para `8.8.8.8`, então o kernel utiliza a rota default.

---

## Destino 3 - Fora da LAN

Destino:

```text
10.0.0.20
```

A rota que vence é:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
```

**Justificativa:** O endereço `10.0.0.20` não pertence à rede `192.168.0.0/24`. Como não existe uma rota mais específica, a rota default é utilizada.

---

# Riscos

Uma rota default incorreta pode causar os seguintes problemas:

1. **Perda de acesso à Internet:** os pacotes podem ser enviados para um gateway incorreto e não chegar ao destino.

2. **Falha de acesso a redes externas:** serviços fora da rede local podem ficar inacessíveis, mesmo que a interface de rede esteja funcionando.

3. **Encaminhamento pelo caminho errado:** em sistemas com várias interfaces, uma rota default incorreta pode fazer o tráfego sair por uma interface errada, como VPN, Wi-Fi ou Ethernet.

# Conclusão

O comando `ip route` permite verificar como o Linux decide para onde enviar os pacotes. Rotas mais específicas têm prioridade sobre a rota default, e a métrica pode ser usada para decidir entre rotas equivalentes.
