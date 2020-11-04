# Deploy Zabbix in Docker Swarm

`The instructions for this repository are being translated into English`

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

- [Deploy MySQL 8](steps/1_deploy_db.md)
- [Deploy Docker Swarm](steps/2_deploy_swarm.md)
- [Deploy NFS](steps/3_deploy_nfs-server_centos.md)
- [Deploy Stack Zabbix](steps/4_deploy_zabbix_on_docker.md)

