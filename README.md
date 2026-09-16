# TuxCare KernelCare e ePortal com Ansible

Laboratorio Ansible dividido em tres fluxos independentes:

1. instalar o agente KernelCare diretamente pela TuxCare Cloud;
2. instalar e configurar um servidor ePortal;
3. instalar os agentes e incluir as maquinas no ePortal.

## Arquitetura

```text
TuxCare Cloud
    |
    | HTTPS: patchsets e metadados
    v
ePortal interno
    |
    | HTTP/HTTPS: instalador, registro e patches
    +--> linux-client-01
    +--> linux-client-02
```

O deploy direto pela Cloud continua disponivel para a VM `tuxcare-lab`.

## Documentacao por etapa

- [1. Deploy do agente em uma maquina](docs/01-deploy-machine.md)
- [2. Deploy do servidor ePortal](docs/02-deploy-eportal.md)
- [3. Inclusao das maquinas no ePortal](docs/03-register-machines-eportal.md)

## Playbooks

| Objetivo | Playbook | Grupo do inventario |
| --- | --- | --- |
| Agente via TuxCare Cloud | `playbooks/deploy-machine.yml` | `kernelcare_cloud` |
| Servidor ePortal | `playbooks/deploy-eportal.yml` | `eportal` |
| Agentes conectados ao ePortal | `playbooks/register-machines-eportal.yml` | `kernelcare_eportal` |
| Estado de todos os agentes | `playbooks/status.yml` | `kernelcare` |

O antigo `playbooks/deploy.yml` foi mantido como atalho para `deploy-machine.yml`.

## Preparacao

```bash
sudo dnf install -y git-core ansible-core
git clone git@github.com:rafaelpedrosorosa/tuxcare-kernelcare-ansible-lab.git
cd tuxcare-kernelcare-ansible-lab
ansible-inventory --graph
```

Copie os arquivos `vault.yml.example` correspondentes para `vault.yml` usando `ansible-vault create`. Os arquivos reais de segredo sao ignorados pelo Git.

## Estrutura principal

```text
inventory/
  hosts.yml
  group_vars/
    eportal/
    kernelcare/
    kernelcare_cloud/
    kernelcare_eportal/
playbooks/
  deploy-machine.yml
  deploy-eportal.yml
  register-machines-eportal.yml
  status.yml
roles/
  tuxcare_kernelcare/
  tuxcare_eportal/
docs/
  01-deploy-machine.md
  02-deploy-eportal.md
  03-register-machines-eportal.md
```

## Ordem recomendada do laboratorio

1. Teste `deploy-machine.yml` na VM local usando a activation key do trial Cloud.
2. Prepare uma VM dedicada e execute `deploy-eportal.yml`.
3. Configure as credenciais de Patch Source no painel do ePortal.
4. Cadastre uma VM canario no grupo `kernelcare_eportal`.
5. Execute `register-machines-eportal.yml` somente no canario.
6. Confirme o check-in no painel e depois registre as demais maquinas.

## Referencias oficiais

- [KernelCare](https://docs.tuxcare.com/live-patching-services/)
- [KernelCare ePortal](https://docs.tuxcare.com/eportal/)
- [ePortal API](https://docs.tuxcare.com/eportal-api/)

