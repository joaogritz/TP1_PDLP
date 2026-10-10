# Hostnames

## 1. Estado inicial

```bash
hostname
```

Saída:

```text
fedora
```

```bash
hostnamectl
```

Saída:

```text
  Transient hostname: fedora
     Static hostname: (unset)
           Icon name: computer-laptop
             Chassis: laptop 💻
   Chassis Asset Tag: NO Asset Tag
          Machine ID: 18ef80cd39c64412b6d114c22217b4de
             Boot ID: ffc04cfc5a38407ea24886e2af1c6065
    Operating System: Fedora Linux 44 (Workstation Edition)
         CPE OS Name: cpe:/o:fedoraproject:fedora:44
      OS Support End: Wed 2027-05-19
OS Support Remaining: 7month 1w
              Kernel: Linux 6.19.10-300.fc44.x86_64
        Architecture: x86-64
     Hardware Vendor: Lenovo
      Hardware Model: IdeaPad Gaming 3 15IHU6
        Hardware SKU: LENOVO_MT_82MG_BU_idea_FM_IdeaPad Gaming 3 15IHU6
    Firmware Version: H4CN28WW(V1.13)
       Firmware Date: Mon 2023-01-09
        Firmware Age: 3y 9month
```

**Conclusão:** O hostname inicial era `fedora`, apresentado como transitório. O hostname estático estava sem definição (`(unset)`).

---

## 2. Convenção de hostname

Foi escolhido `dr4-joao-linux`, seguindo a estrutura `dr4-<usuario>-<papel>`. O prefixo identifica o ambiente, `joao` identifica o responsável e `linux` indica o sistema utilizado.

---

## 3. Alteração do hostname

Primeiro, o nome foi digitado diretamente no terminal:

```bash
dr4-joao-linux
```

Saída:

```text
bash: dr4-joao-linux: command not found...
```

O shell tentou executar o nome como comando. Para alterar o hostname, foi utilizado:

```bash
sudo hostnamectl set-hostname dr4-joao-linux
```

Saída: nenhuma; o comando retornou ao prompt sem mensagens.

Verificação:

```bash
hostname
```

Saída:

```text
dr4-joao-linux
```

```bash
hostnamectl
```

Saída:

```text
     Static hostname: dr4-joao-linux
           Icon name: computer-laptop
             Chassis: laptop 💻
   Chassis Asset Tag: NO Asset Tag
          Machine ID: 18ef80cd39c64412b6d114c22217b4de
             Boot ID: ffc04cfc5a38407ea24886e2af1c6065
    Operating System: Fedora Linux 44 (Workstation Edition)
         CPE OS Name: cpe:/o:fedoraproject:fedora:44
      OS Support End: Wed 2027-05-19
OS Support Remaining: 7month 1w
              Kernel: Linux 6.19.10-300.fc44.x86_64
        Architecture: x86-64
     Hardware Vendor: Lenovo
      Hardware Model: IdeaPad Gaming 3 15IHU6
        Hardware SKU: LENOVO_MT_82MG_BU_idea_FM_IdeaPad Gaming 3 15IHU6
    Firmware Version: H4CN28WW(V1.13)
       Firmware Date: Mon 2023-01-09
        Firmware Age: 3y 9month
```

**Conclusão:** O hostname estático foi definido como `dr4-joao-linux`. A alteração foi confirmada pelos dois comandos; não foi apresentado um teste após reinicialização.

---

## 4. Verificação do /etc/hosts

Antes da edição:

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

O arquivo não continha uma entrada para `fedora`. Foi aberto para edição com:

```bash
sudo nano /etc/hosts
```

Após a edição:

```bash
cat /etc/hosts
```

Saída:

```text
# Loopback entries; do not change.
# For historical reasons, localhost precedes localhost.localdomain:
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         dr4-joao-linux localhost.localdomain localhost6 localhost6.localdomain6
# See hosts(5) for proper format and other examples:
# 192.168.1.10 foo.example.org foo
# 192.168.1.13 bar.example.org bar
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1	kubernetes.docker.internal
# End of section
```

**Conclusão:** A edição associou `dr4-joao-linux` ao endereço de loopback IPv6 `::1`. Ela substituiu o primeiro nome `localhost` nessa linha; a associação de `localhost` a `127.0.0.1` permaneceu na linha IPv4.

---

## 5. Reversão e restauração do arquivo hosts

Foi executado:

```bash
sudo hostnamectl set-hostname fedora
```

Saída: nenhuma; o comando retornou ao prompt sem mensagens.

Verificação:

```bash
hostname
```

Saída:

```text
fedora
```

```bash
hostnamectl
```

Saída:

```text
     Static hostname: fedora
           Icon name: computer-laptop
             Chassis: laptop 💻
   Chassis Asset Tag: NO Asset Tag
          Machine ID: 18ef80cd39c64412b6d114c22217b4de
             Boot ID: ffc04cfc5a38407ea24886e2af1c6065
    Operating System: Fedora Linux 44 (Workstation Edition)
         CPE OS Name: cpe:/o:fedoraproject:fedora:44
      OS Support End: Wed 2027-05-19
OS Support Remaining: 7month 1w
              Kernel: Linux 6.19.10-300.fc44.x86_64
        Architecture: x86-64
     Hardware Vendor: Lenovo
      Hardware Model: IdeaPad Gaming 3 15IHU6
        Hardware SKU: LENOVO_MT_82MG_BU_idea_FM_IdeaPad Gaming 3 15IHU6
    Firmware Version: H4CN28WW(V1.13)
       Firmware Date: Mon 2023-01-09
        Firmware Age: 3y 9month
```

**Conclusão:** O nome voltou a ser `fedora`. O estado inicial não foi reproduzido integralmente: antes, `fedora` era transitório e o hostname estático estava indefinido; ao final, `fedora` passou a ser o hostname estático.

Na primeira coleta do exercício 12, a entrada de teste ainda estava no arquivo. Posteriormente, a linha original foi restaurada com:

```bash
sudo nano /etc/hosts
```

Após salvar e fechar o editor, foi realizada a verificação:

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

**Conclusão da restauração do arquivo:** A associação de `dr4-joao-linux` foi removida e a linha original de `::1` foi restaurada. O nome da máquina permanece `fedora`, configurado como estático; a diferença em relação ao estado inicial transitório está registrada acima.

# Conclusão

As saídas confirmam a alteração para `dr4-joao-linux` e a volta para `fedora`, agora configurado como hostname estático. A coleta final confirma a restauração do conteúdo original de `/etc/hosts`. Alterar o hostname com `hostnamectl` e editar associações no `/etc/hosts` são operações distintas, e ambas precisam ser verificadas ao reverter o teste.
