# Deploy Zabbix Server, Frontend e Grafana no Docker

- [Deploy Zabbix Server, Frontend e Grafana no Docker](#deploy-zabbix-server-frontend-e-grafana-no-docker)
  - [Deploy stack Zabbix](#deploy-stack-zabbix)
    - [Install GIT](#install-git)
    - [Clonando depositório](#clonando-depositório)
    - [Inicianlizando a stack](#inicianlizando-a-stack)
    - [Listando stacks disponiveis](#listando-stacks-disponiveis)
    - [Listando status dos serviços](#listando-status-dos-serviços)
    - [Verificar logs](#verificar-logs)
    - [Removendo stack](#removendo-stack)


## Deploy stack Zabbix

### Install GIT

```bash
dnf install -y git
```

### Clonando depositório

```bash
cd ~/
git clone https://github.com/robertsilvatech/deploy-zabbix-dockerswarm.git
```

### Inicianlizando a stack 

```bash
cd deploy-zabbix-dockerswarm
docker stack deploy -c docker-compose.yaml workshop
sh create.sh
```

### Listando stacks disponiveis

```bash
docker stack ls
```

### Listando status dos serviços

```bash
docker service ls
```

### Verificar logs

```bash
docker service logs -f workshop_zabbix-server
docker service logs -f workshop_zabbix-frontend
```
 
### Removendo stack

```bash
docker stack rm maratonazabbix
```

