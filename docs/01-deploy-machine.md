# 1. Deploy do agente em uma maquina

Este fluxo instala o KernelCare diretamente pela TuxCare Cloud em uma VM RHEL, AlmaLinux ou Rocky Linux 9.

## Preparar a VM

```bash
sudo dnf install -y git-core ansible-core
git clone git@github.com:rafaelpedrosorosa/tuxcare-kernelcare-ansible-lab.git
cd tuxcare-kernelcare-ansible-lab
```

O host de laboratorio esta no grupo `kernelcare_cloud` e usa conexao local:

```yaml
kernelcare_cloud:
  hosts:
    tuxcare-lab:
      ansible_connection: local
      ansible_host: 127.0.0.1
```

## Armazenar a activation key do trial

```bash
ansible-vault create inventory/group_vars/kernelcare_cloud/vault.yml
```

Conteudo:

```yaml
---
vault_tuxcare_activation_key: "CHAVE_DO_TRIAL_TUXCARE_CLOUD"
```

O arquivo real `vault.yml` e ignorado pelo Git. Somente o exemplo pode ser versionado.

## Validar e executar

```bash
ansible-inventory --graph
ansible kernelcare_cloud -m ansible.builtin.ping

ansible-playbook playbooks/deploy-machine.yml \
  --syntax-check \
  --ask-vault-pass

ansible-playbook playbooks/deploy-machine.yml \
  --ask-vault-pass \
  --ask-become-pass
```

## Conferir

```bash
sudo kcarectl --info
sudo kcarectl --patch-info
sudo kcarectl --uname
sudo kcarectl --license-info
ansible-playbook playbooks/status.yml
```

