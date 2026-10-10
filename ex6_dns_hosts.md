# DNS e hosts

## 1. Arquivo /etc/hosts

Comando utilizado:

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

**Conclusão:** O arquivo `/etc/hosts` associa os nomes de localhost aos endereços de loopback `127.0.0.1` e `::1`. Também contém a entrada `kubernetes.docker.internal` apontando para `127.0.0.1`, adicionada pelo Docker Desktop.

---

## 2. Configuração do resolvedor

Comando utilizado:

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

**Conclusão:** O sistema utiliza o resolvedor local do `systemd-resolved` em `127.0.0.53`. Para consultar os servidores DNS utilizados na conexão, o próprio arquivo indica o comando `resolvectl status`.

---

## 3. Adicionando um nome fictício

Foi utilizado o nome:

```text
servico-interno.local
```

Para adicionar a entrada no `/etc/hosts`:

```bash
echo "127.0.0.1 servico-interno.local" | sudo tee -a /etc/hosts
```

Saída:

```text
127.0.0.1 servico-interno.local
```

Depois, para verificar:

```bash
getent hosts servico-interno.local
```

Saída:

```text
127.0.0.1       servico-interno.local
```

**Conclusão:** O nome `servico-interno.local` passou a resolver para `127.0.0.1` através do arquivo `/etc/hosts`.

---

## 4. Removendo a entrada

Para remover a entrada:

```bash
sudo sed -i '/servico-interno.local/d' /etc/hosts
```

Saída: nenhuma; o comando retornou ao prompt sem exibir mensagens.

Depois, execute novamente:

```bash
getent hosts servico-interno.local
```

Saída: nenhuma; a consulta não retornou registros.

**Conclusão:** Após remover a entrada do `/etc/hosts`, o nome deixou de ser resolvido.

---

# Diferença entre /etc/hosts e DNS

O arquivo `/etc/hosts` realiza uma associação local entre um nome e um endereço IP.  
Essa configuração funciona apenas na máquina onde o arquivo foi alterado.  
O DNS, por outro lado, utiliza servidores de nomes para resolver endereços de forma centralizada.  
Com DNS, vários dispositivos podem consultar a mesma informação.  
O `/etc/hosts` pode ser útil para testes, desenvolvimento ou pequenas correções temporárias.  
Porém, alterações precisam ser feitas manualmente em cada máquina.  
Por isso, o DNS é mais adequado para ambientes maiores e para configurações permanentes.

---

# Riscos do uso permanente de /etc/hosts

1. **Informações desatualizadas:** caso o IP de um serviço mude, a entrada no `/etc/hosts` pode continuar apontando para o endereço antigo.

2. **Dificuldade de manutenção:** em ambientes com várias máquinas, seria necessário alterar manualmente o arquivo `/etc/hosts` de cada uma delas.

# Conclusão

O `/etc/hosts` permite realizar overrides locais de resolução de nomes de forma simples. Entretanto, ele deve ser utilizado principalmente para testes ou situações controladas, enquanto o DNS é mais adequado para resolução de nomes em ambientes de produção.