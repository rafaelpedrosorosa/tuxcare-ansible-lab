# TuxCare com Ansible: Radar, KernelCare e ePortal

Laboratorio Ansible organizado em fases independentes. A implementacao inicial
e o **TuxCare Radar**, usado para descoberta e avaliacao de vulnerabilidades.
KernelCare Enterprise e ePortal permanecem no repositorio para as fases
seguintes.

## Ordem do laboratorio

1. **Radar:** instalar `tuxcare-radar`, publicar `radar.yaml` e executar o
   primeiro scan.
2. **KernelCare Enterprise:** instalar e registrar o agente de live patching.
3. **ePortal:** implantar o servidor interno e redirecionar os clientes.

## Sistemas atendidos pelo Radar

- RPM: AlmaLinux, CentOS Stream e RHEL.
- APT: Ubuntu e Debian.

O role detecta `ansible_os_family` e configura automaticamente o repositorio
DNF ou APT documentado pela TuxCare. A disponibilidade para cada versao e
arquitetura depende dos pacotes publicados pelo fabricante.

## Playbooks

| Fase | Objetivo | Playbook | Grupo |
| --- | --- | --- | --- |
| 1 | Instalar e configurar Radar | `playbooks/deploy-radar.yml` | `radar_clients` |
| 2 | Agente via TuxCare Cloud | `playbooks/deploy-machine.yml` | `kernelcare_cloud` |
| 3 | Servidor ePortal | `playbooks/deploy-eportal.yml` | `eportal` |
| 3 | Agentes conectados ao ePortal | `playbooks/register-machines-eportal.yml` | `kernelcare_eportal` |
| Apoio | Estado dos agentes KernelCare | `playbooks/status.yml` | `kernelcare` |

O antigo `playbooks/deploy.yml` foi mantido como atalho para
`deploy-machine.yml`.

## Inicio rapido: Radar

```bash
sudo dnf install -y git-core ansible-core
git clone https://github.com/rafaelpedrosorosa/tuxcare-ansible-lab.git
cd tuxcare-ansible-lab

cp inventory/group_vars/radar_clients/vault.yml.example \
  inventory/group_vars/radar_clients/vault.yml
# Edite vault.yml e informe a API key do Radar antes de criptografar.
ansible-vault encrypt inventory/group_vars/radar_clients/vault.yml

ansible-inventory --graph
ansible-playbook playbooks/deploy-radar.yml --ask-vault-pass
```

O arquivo real `vault.yml` e ignorado pelo Git. Nunca versione a API key ou a
senha do Vault em texto claro.

## Documentacao por fase

- [Fase 1: deploy do TuxCare Radar](docs/00-deploy-radar.md)
- [Fase 2: deploy do KernelCare em uma maquina](docs/01-deploy-machine.md)
- [Fase 3: deploy do servidor ePortal](docs/02-deploy-eportal.md)
- [Fase 3: inclusao das maquinas no ePortal](docs/03-register-machines-eportal.md)

## Estrutura principal

```text
inventory/
  hosts.yml
  group_vars/
    radar_clients/
    kernelcare_cloud/
    kernelcare_eportal/
    eportal/
playbooks/
  deploy-radar.yml
  deploy-machine.yml
  deploy-eportal.yml
  register-machines-eportal.yml
roles/
  tuxcare_radar/
  tuxcare_kernelcare/
  tuxcare_eportal/
docs/
  00-deploy-radar.md
  01-deploy-machine.md
  02-deploy-eportal.md
  03-register-machines-eportal.md
```

## O que o deploy do Radar faz

1. valida a familia do sistema e a presenca da API key;
2. configura o repositorio oficial TuxCare para RPM ou APT;
3. instala somente o pacote `tuxcare-radar`;
4. publica `/etc/tuxcare-radar/radar.yaml` com modo `0600`;
5. preserva o agendamento criado pelo proprio pacote.

O template e unico para todas as distribuicoes. A API key vem do Ansible Vault,
e a task que grava a configuracao usa `no_log: true`.

## Referencias oficiais

- [TuxCare Radar](https://docs.tuxcare.com/radar/)
- [KernelCare](https://docs.tuxcare.com/live-patching-services/)
- [KernelCare ePortal](https://docs.tuxcare.com/eportal/)
- [ePortal API](https://docs.tuxcare.com/eportal-api/)

