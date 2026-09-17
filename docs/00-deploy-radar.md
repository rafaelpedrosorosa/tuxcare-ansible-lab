# Fase 1: deploy do TuxCare Radar

Este fluxo instala somente o pacote `tuxcare-radar` e publica o arquivo
`/etc/tuxcare-radar/radar.yaml`. O mesmo playbook atende:

- AlmaLinux, CentOS Stream e RHEL, usando DNF;
- Ubuntu e Debian, usando APT.

O role detecta a familia do sistema pelos facts do Ansible.

## 1. Definir os hosts

O grupo `radar_clients` fica em `inventory/hosts.yml`. O host `tuxcare-lab` ja
esta associado ao grupo e reutiliza a conexao local definida no inventario.

Para servidores remotos, substitua ou acrescente hosts:

```yaml
radar_clients:
  hosts:
    alma9:
      ansible_host: 192.168.56.61
      ansible_user: ansible
    ubuntu2404:
      ansible_host: 192.168.56.62
      ansible_user: ansible
```

## 2. Proteger a API key com Ansible Vault

Crie o arquivo real a partir do exemplo:

```bash
cp inventory/group_vars/radar_clients/vault.yml.example \
  inventory/group_vars/radar_clients/vault.yml

ansible-vault encrypt inventory/group_vars/radar_clients/vault.yml
```

Antes de criptografar, substitua o valor de exemplo:

```yaml
---
vault_tuxcare_radar_api_key: "SUA_API_KEY_DO_RADAR"
```

O `vault.yml` real esta no `.gitignore`. Nao versione a API key em texto claro
nem a senha usada para abrir o Vault.

## 3. Validar e executar

```bash
ansible-inventory --graph
ansible radar_clients -m ansible.builtin.ping

ansible-playbook playbooks/deploy-radar.yml \
  --syntax-check \
  --ask-vault-pass

ansible-playbook playbooks/deploy-radar.yml \
  --ask-vault-pass \
  --ask-become-pass
```

## 4. Conferir no host

```bash
rpm -q tuxcare-radar       # AlmaLinux/CentOS/RHEL
dpkg -s tuxcare-radar      # Ubuntu/Debian
sudo test -s /etc/tuxcare-radar/radar.yaml
sudo su -s /bin/bash nobody -c \
  "tuxcare-radar --config /etc/tuxcare-radar/radar.yaml"
```

O pacote cria `/etc/cron.d/tuxcare-radar`. O cron verifica se o intervalo de
`time-between-runs` ja passou; neste projeto, o valor padrao e `23h`.

O `radar.yaml` contem a API key em texto claro no host. Por isso, ele e criado
com modo `0600`, e a task do template usa `no_log: true`. Evite exibir esse
arquivo em evidencias, logs ou pipelines.

## Opcoes

As opcoes nao sensiveis estao em
`inventory/group_vars/radar_clients/main.yml`.

Para a instancia europeia:

```yaml
tuxcare_radar_base_url: "https://eu.radar.tuxcare.com"
```

Antes de clonar uma VM, a TuxCare recomenda zerar `/etc/machine-id`. Para
clones existentes que compartilham esse identificador, habilite:

```yaml
tuxcare_radar_use_alternative_node_id: true
```

## Idempotencia

Execute o playbook uma segunda vez. O esperado e `changed=0`, exceto quando
houver atualizacao de pacote ou de metadados do gerenciador.
