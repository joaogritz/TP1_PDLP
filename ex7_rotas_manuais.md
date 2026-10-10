# Rotas manuais

## 1. Estado inicial

Comando utilizado:

```bash
ip route
```

Saída real:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** O host possui uma rota default pelo gateway `192.168.0.1`, utilizando a interface Wi-Fi `wlp0s20f3`, e uma rota conectada para `192.168.0.0/24`.

---

## 2. Adição de uma rota manual

Foi escolhido o endereço `8.8.8.8`, utilizando `/32` para limitar a rota a esse único IP.

### Tentativa realizada

```bash
sudo ip route add 8.8.8.8/32 via 192.168.1.1 dev eth0
```

Saída real:

```text
[sudo] password for joaopogo:
Cannot find device "eth0"
```

**Conclusão:** A tentativa falhou porque a interface `eth0` não existe na máquina. Esse comando não adicionou a rota. O gateway informado também difere do gateway da conexão atual.

### Comando corrigido para esta máquina

```bash
sudo ip route add 8.8.8.8/32 via 192.168.0.1 dev wlp0s20f3
```

Saída real:

```text
[sudo] password for joaopogo:
```

Após solicitar a senha, o comando retornou ao prompt sem mensagens de erro.

O gateway é o mesmo da rota default. Portanto, a rota específica muda a seleção da rota para esse destino, mas mantém o mesmo próximo salto e a mesma interface.

---

## 3. Verificação da rota

Após executar o comando corrigido:

```bash
ip route
```

Saída real:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
8.8.8.8 via 192.168.0.1 dev wlp0s20f3
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** A tabela confirma a adição da rota específica para `8.8.8.8` via `192.168.0.1`, pela interface `wlp0s20f3`. A rota `/32` tem prioridade sobre a rota default para esse destino por ser mais específica.

---

# Justificativa do uso de /32

O prefixo `/32` representa um único endereço IPv4. Assim, a rota se aplica somente ao tráfego destinado a `8.8.8.8`.

Um prefixo menos específico, como `/24`, abrange 256 endereços e pode alterar o encaminhamento para outros destinos. O uso de `/32` limita o alcance do teste.

---

## 4. Remoção da rota

Após confirmar a adição, foi executada a remoção:

```bash
sudo ip route del 8.8.8.8/32 via 192.168.0.1 dev wlp0s20f3
```

Saída real: nenhuma. O comando retornou ao prompt sem exibir mensagens.

Depois:

```bash
ip route
```

Saída real após a remoção:

```text
default via 192.168.0.1 dev wlp0s20f3 proto dhcp src 192.168.0.12 metric 600
192.168.0.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.0.12 metric 600
```

**Conclusão:** A rota específica para `8.8.8.8` deixou de aparecer na tabela, confirmando sua remoção. A tabela voltou ao estado inicial.

---

# Plano de rollback

1. Identificar a rota adicionada utilizando `ip route`.
2. Remover a rota do teste com `sudo ip route del 8.8.8.8/32 via 192.168.0.1 dev wlp0s20f3`.
3. Executar novamente `ip route` e testar a conectividade com `ping -c 4 8.8.8.8` para verificar o estado após a remoção.

# Conclusão

As saídas reais confirmam a adição da rota `8.8.8.8/32` via `192.168.0.1` na interface `wlp0s20f3` e sua posterior remoção, restaurando a tabela inicial. A tentativa anterior com `eth0` falhou e foi corrigida utilizando a interface e o gateway reais da máquina.

Rotas manuais podem ser úteis para diagnóstico. Neste teste, o prefixo `/32` limita a seleção da rota a um único destino, e a remoção da entrada permite retornar à tabela inicial.
