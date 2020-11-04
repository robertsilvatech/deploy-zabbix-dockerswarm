# Deploy Zabbix Server, Frontend e Grafana no Docker

- [Deploy Zabbix Server, Frontend e Grafana no Docker](#deploy-zabbix-server-frontend-e-grafana-no-docker)
  - [Primeiro passo corrigir o horário do servidor](#primeiro-passo-corrigir-o-horário-do-servidor)
    - [Verificar o timezone atual](#verificar-o-timezone-atual)
    - [Listar timezone disponiveis](#listar-timezone-disponiveis)
    - [Definir timezone](#definir-timezone)
    - [Verifique status do NTP](#verifique-status-do-ntp)
  - [Ajustes de S.O após a instalação](#ajustes-de-so-após-a-instalação)
    - [Verificar se é necessário atualização do S.O](#verificar-se-é-necessário-atualização-do-so)
    - [Instalar utilitários](#instalar-utilitários)
  - [Instalando o Docker](#instalando-o-docker)
    - [Instalando repositório do docker](#instalando-repositório-do-docker)
    - [Removendo repositórios antigos](#removendo-repositórios-antigos)
    - [Listando versões disponiveis do docker](#listando-versões-disponiveis-do-docker)
    - [Instalando device mapper](#instalando-device-mapper)
    - [Instalando a ultima versão do Docker](#instalando-a-ultima-versão-do-docker)
    - [Inicializando o daemon do docker](#inicializando-o-daemon-do-docker)
    - [Verificando versão do docker](#verificando-versão-do-docker)
    - [Habilitar roteamento nos containers](#habilitar-roteamento-nos-containers)
    - [Ativando modo swarm no host](#ativando-modo-swarm-no-host)
    - [Inspect de todas as redes do docker](#inspect-de-todas-as-redes-do-docker)
    - [Validando conflito de rede](#validando-conflito-de-rede)
      - [Criando uma rede ingress com range diferente](#criando-uma-rede-ingress-com-range-diferente)
        - [Listando os nodes](#listando-os-nodes)
        - [Alterando disponibilidade do node no swarm](#alterando-disponibilidade-do-node-no-swarm)
        - [Removendo rede ingress atual](#removendo-rede-ingress-atual)
        - [Criando nova rede ingress](#criando-nova-rede-ingress)
        - [Ativando a disponibilidade do node novamente](#ativando-a-disponibilidade-do-node-novamente)
    - [Criando rede para o nosso ambiente](#criando-rede-para-o-nosso-ambiente)
    - [Inspect de todas as redes do docker novamente](#inspect-de-todas-as-redes-do-docker-novamente)
  - [Adicionando outros nodes manager no cluster](#adicionando-outros-nodes-manager-no-cluster)
    - [Pegando Token para manager](#pegando-token-para-manager)
    - [Criando regra de firewall em todos os nodes](#criando-regra-de-firewall-em-todos-os-nodes)
  - [Deploy stack Zabbix](#deploy-stack-zabbix)
    - [Instalar o GIT](#instalar-o-git)
    - [Clonando depositório](#clonando-depositório)
    - [Inicianlizando a stack](#inicianlizando-a-stack)
    - [Listando stacks disponiveis](#listando-stacks-disponiveis)
    - [Listando status dos serviços](#listando-status-dos-serviços)
    - [Verificar logs](#verificar-logs)
    - [Removendo stack](#removendo-stack)

## Primeiro passo corrigir o horário do servidor

### Verificar o timezone atual

```bash
timedatectl status
```

Output:

```bash
               Local time: Wed 2020-06-03 05:03:25 -03
           Universal time: Wed 2020-06-03 08:03:25 UTC
                 RTC time: Wed 2020-06-03 08:03:24
                Time zone: America/New_York (-03, -0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

### Listar timezone disponiveis

```bash
timedatectl list-timezones | grep Sao_Paulo
```

Output:

```bash
America/Sao_Paulo
```

### Definir timezone

```bash
timedatectl set-timezone America/Sao_Paulo
```

### Verifique status do NTP

```bash
timedatectl status
```

```bash
               Local time: Wed 2020-06-03 05:03:25 -03
           Universal time: Wed 2020-06-03 08:03:25 UTC
                 RTC time: Wed 2020-06-03 08:03:24
                Time zone: America/Sao_Paulo (-03, -0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## Ajustes de S.O após a instalação

### Verificar se é necessário atualização do S.O

```bash
dnf clean all
dnf check-update
```

### Instalar utilitários

```bash
dnf install -y net-tools vim nano epel-release wget curl tcpdump
```

## Instalando o Docker

### Instalando repositório do docker

```bash
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

### Removendo repositórios antigos

```bash
dnf clean all
```

### Listando versões disponiveis do docker

```bash
dnf list docker-ce --showduplicates | sort -r
```

### Instalando device mapper

```bash
dnf install -y device-mapper-persistent-data
```

### Instalando a ultima versão do Docker

`Se precisar da ultima versão do docker, é necessário instalar manualmente a ultima versão do containerd.io`

```bash
dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
dnf install -y docker-ce
```

`Se não, utilize o parâmetro --nobest na hora de instalação`

```bash
dnf install -y docker-ce --nobest
```

### Inicializando o daemon do docker

```bash
systemctl enable --now docker
systemctl status docker
```

### Verificando versão do docker

```bash
docker version
```

### Habilitar roteamento nos containers

```bash
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --reload
```

### Ativando modo swarm no host

```bash
docker swarm init --advertise-addr 10.0.0.51
```

### Inspect de todas as redes do docker

```bash
for net in `docker network ls |grep -v NAME | awk '{print $2}'`;do ipam=`docker network inspect $net --format {{.IPAM}}` && echo $net - $ipam; done
```

### Validando conflito de rede

```bash
ifconfig ens18
```

Output:

```bash
ens18: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.34  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::d440:d1d9:33cf:3800  prefixlen 64  scopeid 0x20<link>
        inet6 fe80::b5c3:70cc:d7a0:5d57  prefixlen 64  scopeid 0x20<link>
        inet6 fe80::5b2:4fc5:e2cb:dbab  prefixlen 64  scopeid 0x20<link>
        ether ee:2d:73:2e:4b:9f  txqueuelen 1000  (Ethernet)
        RX packets 69824  bytes 177640772 (169.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 32110  bytes 2666365 (2.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### Criando uma rede ingress com range diferente

##### Listando os nodes

```bash
docker node ls
```

##### Alterando disponibilidade do node no swarm

`NESTE MOMENTO ELE NÃO IRA INICIAR NOVOS SERVIÇOS`

`<NODE_NAME> É O NOME DO HOSTNAME QUE RETORNA NO COMANDO docker node ls`

```bash
docker node update --availability drain <NODE_NAME>
```

##### Removendo rede ingress atual

```bash
$ docker network rm ingress
```

Output:

```bash
WARNING! Before removing the routing-mesh network, make sure all the nodes
in your swarm run the same docker engine version. Otherwise, removal may not
be effective and functionality of newly created ingress networks will be
impaired.
Are you sure you want to continue? [y/N]
```

##### Criando nova rede ingress

```bash
docker network create \
--driver overlay \
--ingress \
--subnet=192.168.102.0/28 \
--gateway=192.168.102.2 \
--opt com.docker.network.driver.mtu=1200 \
ingress
```

##### Ativando a disponibilidade do node novamente

`<NODE_NAME> É O NOME DO HOSTNAME QUE RETORNA NO COMANDO docker node ls`

```bash
docker node update --availability active <NODE_NAME>
```

### Criando rede para o nosso ambiente

```bash
docker network create --driver overlay monitoring-network
```

### Inspect de todas as redes do docker novamente

```bash
for net in `docker network ls |grep -v NAME | awk '{print $2}'`;do ipam=`docker network inspect $net --format {{.IPAM}}` && echo $net - $ipam; done
```

## Adicionando outros nodes manager no cluster

### Pegando Token para manager

```bash
docker swarm join-token manager
```

### Criando regra de firewall em todos os nodes

```bash
firewall-cmd --parmanent --add-port=2377/tcp
firewall-cmd --permanent --add-port=7946/tcp
firewall-cmd --permanent --add-port=7946/udp
firewall-cmd --permanent --add-port=4789/udp
firewall-cmd --reload
```

## Deploy stack Zabbix

### Instalar o GIT

```bash
dnf install -y git
```

### Clonando depositório

```bash
cd ~/
git clone <URL DO SEU GIT>
```

`git@github.com:robertsilvatech/deploy-zabbix-docker-advanced.git`

### Inicianlizando a stack 

```bash
cd <NOME_DA_PASTA_CLONADA>
sh grafana.sh
docker stack deploy -c docker-compose.yaml maratonazabbix
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
docker service logs -f maratonazabbix_zabbix-server
docker service logs -f maratonazabbix_zabbix-frontend
docker service logs -f maratonazabbix_grafana
```
 
### Removendo stack

```bash
docker stack rm maratonazabbix
```

