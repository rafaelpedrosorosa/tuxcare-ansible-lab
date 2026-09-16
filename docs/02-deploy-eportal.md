# 2. Deploy do servidor ePortal

O ePortal centraliza chaves, agentes, feeds e a liberacao dos patchsets. Para o laboratorio, utilize uma VM dedicada RHEL, AlmaLinux ou Rocky Linux 8/9.

> Antes da primeira instalacao, confirme com a TuxCare a liberacao do ePortal e solicite as credenciais de Patch Source. A activation key do trial Cloud nao substitui essas credenciais.

## Requisitos sugeridos para o laboratorio

- uma VM EL 8/9;
- DNS ou entrada em `/etc/hosts` para o nome do ePortal;
- acesso dos clientes ao ePortal em TCP 80 no teste ou TCP 443 com TLS;
- acesso de saida do ePortal em TCP 443;
- SSD. O modo cache reduz significativamente o espaco necessario.

O inventario de exemplo utiliza:

```yaml
eportal:
  hosts:
    eportal-lab:
      ansible_host: 192.168.56.50
      ansible_user: ansible
```

Ajuste `inventory/group_vars/eportal/main.yml`:

```yaml
eportal_allowed_hosts:
  - "192.168.56.50"
  - "eportal-lab"

eportal_cache_mode: true
eportal_proxy_url: ""
eportal_admin_user: admin
eportal_registration_key_server_limit: 10
```

## Criar o Vault do ePortal

```bash
ansible-vault create inventory/group_vars/eportal/vault.yml
```

Conteudo:

```yaml
---
vault_eportal_admin_password: "SENHA_FORTE_DO_ADMIN"
vault_eportal_registration_key: "CHAVE_FORTE_PARA_OS_AGENTES"
```

## Instalar

```bash
ansible eportal -m ansible.builtin.ping

ansible-playbook playbooks/deploy-eportal.yml \
  --syntax-check \
  --ask-vault-pass

ansible-playbook playbooks/deploy-eportal.yml \
  --ask-vault-pass \
  --ask-become-pass
```

A role configura os repositorios oficiais, instala `kcare-eportal`, gerencia `/etc/eportal/config`, habilita Nginx/ePortal, cria o administrador e cria a chave usada pelos agentes.

## Configuracao obrigatoria apos o deploy

1. Acesse `http://192.168.56.50/admin` no laboratorio.
2. Entre com o administrador criado pelo playbook.
3. Abra `Settings` > `Patch Source`.
4. Informe as credenciais de Patch Source fornecidas pela TuxCare.
5. Selecione somente as distribuicoes usadas no laboratorio.
6. Confirme a sincronizacao dos metadados/patchsets.

## Firewall e destinos externos

Clientes para ePortal:

```text
TCP 80  - somente laboratorio HTTP
TCP 443 - recomendado com HTTPS
```

ePortal para Internet em TCP 443:

```text
downloads.kernelcare.com
repo.cloudlinux.com
patches.kernelcare.com
```

Em producao, publique o ePortal com HTTPS e instale a CA correspondente nos clientes.

## Diagnostico

```bash
sudo systemctl status eportal nginx
sudo journalctl -u eportal
sudo kc.eportal doctor --no-send
sudo kc.eportal user
sudo kc.eportal key
```

