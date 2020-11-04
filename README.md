# Deploy Zabbix in Docker Swarm

## Requirements

- 1 VM MySQL
  - SELINUX disabled
  - Firewall enabled
- 3 VM Docker HOST
  - SELINUX enabled
  - Firewall enabled
- 1 VM NFS
  - SELINUX disabled
  - Firewall enabled
- 1 VM HAPROXY
  - SELINUX disabled
  - Firewall enabled

## Our environment

database - 10.0.0.61
docker01 - 10.0.0.62
docker02 - 10.0.0.63
docker03 - 10.0.0.64
nfs - 10.0.0.65
haproxy - 10.0.0.66


# Step by step to deploy Zabbix in Docker Swarm

- [Deploy MySQL 8](steps/deploy_db.md)
- [Deploy Docker Swarm](steps/deploy_swarm.md)


Segunda Maratona Zabbix

Este repositório contem os arquivos da segunda maratona Zabbix.

Para fazer o deploy do ambiente siga os seguintes procedimentos

- [Deploy MySQL 8 no Centos 8](procedimentos/deploy_db.md)
- [Deploy Stack Zabbix em Docker com Centos 8](procedimentos/deploy_zabbix_front_grafana_on_docker.md)