# 3. Inclusao das maquinas no ePortal

Este fluxo instala o agente KernelCare nos clientes, configura o endereco do ePortal e registra cada maquina usando a chave criada no deploy do servidor.

## Resolver o endereco do ePortal

Todos os clientes devem resolver e acessar o endereco configurado em `tuxcare_eportal_url`. No laboratorio de exemplo:

```yaml
tuxcare_eportal_url: "http://192.168.56.50"
```

Em producao, prefira:

```yaml
tuxcare_eportal_url: "https://eportal.empresa.local"
```

## Cadastrar os clientes no inventario

Edite `inventory/hosts.yml`:

```yaml
kernelcare_eportal:
  hosts:
    linux-client-01:
      ansible_host: 192.168.56.61
      ansible_user: ansible
    linux-client-02:
      ansible_host: 192.168.56.62
      ansible_user: ansible
```

## Configurar as variaveis dos clientes

Arquivo `inventory/group_vars/kernelcare_eportal/main.yml`:

```yaml
---
tuxcare_source: eportal
tuxcare_eportal_url: "http://192.168.56.50"
tuxcare_registration_id: "eportal-lab-key-v1"
```

Crie o Vault:

```bash
ansible-vault create inventory/group_vars/kernelcare_eportal/vault.yml
```

Use exatamente a mesma chave informada em `vault_eportal_registration_key` durante o deploy do ePortal:

```yaml
---
vault_tuxcare_activation_key: "CHAVE_DE_REGISTRO_GERADA_NO_EPORTAL"
```

## Testar conectividade

```bash
ansible kernelcare_eportal -m ansible.builtin.ping

ansible kernelcare_eportal -b -m ansible.builtin.uri -a \
  'url=http://192.168.56.50/install-kernelcare return_content=no status_code=200'
```

## Registrar primeiro uma maquina canario

```bash
ansible-playbook playbooks/register-machines-eportal.yml \
  --limit linux-client-01 \
  --ask-vault-pass \
  --ask-become-pass
```

No painel do ePortal, confirme que `linux-client-01` apareceu e realizou check-in.

## Registrar o restante em lotes

```bash
ansible-playbook playbooks/register-machines-eportal.yml \
  --limit kernelcare_eportal \
  --ask-vault-pass \
  --ask-become-pass \
  -e rollout_batch=1
```

A role grava nos clientes:

```ini
AUTO_UPDATE=True
PATCH_SERVER=http://192.168.56.50/
REGISTRATION_URL=http://192.168.56.50/admin/api/kcare
```

Depois executa:

```bash
kcarectl --register CHAVE_DO_EPORTAL
kcarectl --update
kcarectl --status
```

## Validar em cada cliente

```bash
sudo cat /etc/sysconfig/kcare/kcare.conf
sudo kcarectl --license-info
sudo kcarectl --info
sudo kcarectl --status
```

Com `AUTO_UPDATE=True`, o agente consulta o ePortal periodicamente. O ePortal controla qual patchset sera oferecido por meio dos feeds.

