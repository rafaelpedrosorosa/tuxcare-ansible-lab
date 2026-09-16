# Laboratorio TuxCare KernelCare com Ansible

Laboratorio para instalar, registrar e validar o KernelCare em uma unica VM RHEL, AlmaLinux ou Rocky Linux 9. O Ansible e executado dentro da propria VM usando conexao local.

## O que este projeto faz

- baixa o instalador oficial do KernelCare por HTTPS;
- instala um agente na VM local;
- registra a VM usando uma activation key protegida pelo Ansible Vault;
- habilita a verificacao automatica de patches;
- aplica imediatamente os patches disponiveis;
- valida o resultado usando `kcarectl --status`.

## 1. Preparar a VM

Entre na VM e instale Git e Ansible Core:

```bash
sudo dnf install -y git-core ansible-core
ansible-playbook --version
git --version
```

Se o pacote `ansible-core` nao estiver disponivel, confirme se os repositorios BaseOS e AppStream da distribuicao estao habilitados.

Clone este repositorio privado pela sua chave SSH do GitHub:

```bash
git clone git@github.com:rafaelpedrosorosa/tuxcare-kernelcare-ansible-lab.git
cd tuxcare-kernelcare-ansible-lab
```

Se preferir HTTPS, o GitHub exige um personal access token no lugar da senha da conta:

```bash
git clone https://github.com/rafaelpedrosorosa/tuxcare-kernelcare-ansible-lab.git
cd tuxcare-kernelcare-ansible-lab
```

## 2. Obter a chave de teste

Solicite o trial do KernelCare no site da TuxCare. Voce recebera uma activation key. Nao grave essa chave diretamente em playbooks, commits ou no README.

## 3. Criar o segredo criptografado

Na raiz do repositorio, execute:

```bash
ansible-vault create inventory/group_vars/kernelcare/vault.yml
```

No editor aberto pelo Vault, coloque:

```yaml
---
vault_tuxcare_activation_key: "SUA_CHAVE_DE_TESTE"
```

O arquivo `vault.yml` esta no `.gitignore`. Para um repositorio compartilhado, tambem e possivel versionar o arquivo ja criptografado, mas nunca a chave em texto puro.

## 4. Validar antes da instalacao

```bash
ansible-inventory --graph
ansible kernelcare -m ansible.builtin.ping
ansible-playbook playbooks/deploy.yml --syntax-check --ask-vault-pass
```

O inventario usa:

```yaml
tuxcare-lab:
  ansible_connection: local
  ansible_host: 127.0.0.1
```

Portanto, SSH nao e necessario neste laboratorio.

## 5. Instalar o KernelCare

Se seu usuario utiliza sudo com senha:

```bash
ansible-playbook playbooks/deploy.yml \
  --ask-vault-pass \
  --ask-become-pass
```

Se estiver conectado como `root`, retire `--ask-become-pass`.

## 6. Conferir o funcionamento

```bash
ansible-playbook playbooks/status.yml
sudo kcarectl --info
sudo kcarectl --patch-info
sudo kcarectl --uname
sudo kcarectl --license-info
```

Codigos retornados por `kcarectl --status`:

| Codigo | Significado |
| ---: | --- |
| 0 | Atualizado no patchset mais recente |
| 1 | Nenhum patch aplicado |
| 2 | Existem patches novos ainda nao aplicados |
| 3 | Kernel nao suportado |

Com `AUTO_UPDATE=True`, o agente verifica periodicamente se existem novos patches. A role tambem executa `kcarectl --update` na primeira implantacao.

## 7. Versionar as proximas alteracoes

Antes do primeiro commit, confira se a chave nao sera incluida:

```bash
git status --short
git check-ignore -v inventory/group_vars/kernelcare/vault.yml
```

Depois de alterar o laboratorio, revise e envie ao repositorio:

```bash
git add .
git commit -m "descricao objetiva da alteracao"
git push -u origin main
```

## Rotacionar a activation key

Edite o segredo:

```bash
ansible-vault edit inventory/group_vars/kernelcare/vault.yml
```

Depois altere `tuxcare_registration_id` em `inventory/group_vars/kernelcare/main.yml`, por exemplo de `trial-key-v1` para `trial-key-v2`. Esse identificador nao contem o segredo; ele apenas informa a role que deve registrar novamente a VM.

## Remover o laboratorio

```bash
sudo kcarectl --unregister
sudo dnf remove -y kernelcare
```
