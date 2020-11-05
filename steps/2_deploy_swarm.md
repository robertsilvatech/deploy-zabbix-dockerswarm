# Deploy SWARM

- [Deploy SWARM](#deploy-swarm)
  - [Timezone](#timezone)
    - [Check the timezone](#check-the-timezone)
    - [Set timezone](#set-timezone)
  - [Tuning S.O](#tuning-so)
    - [Check updates](#check-updates)
    - [Install utilities](#install-utilities)
  - [Installing Docker](#installing-docker)
    - [Installing the docker repository](#installing-the-docker-repository)
    - [Removing old repositories](#removing-old-repositories)
    - [Installing device mapper](#installing-device-mapper)
    - [Installing the latest version of Docker](#installing-the-latest-version-of-docker)
    - [Initializing the docker daemon](#initializing-the-docker-daemon)
    - [Checking docker version](#checking-docker-version)
    - [Enable routing in containers](#enable-routing-in-containers)
    - [Ativando modo swarm no host](#ativando-modo-swarm-no-host)
    - [Inspect all docker networks](#inspect-all-docker-networks)
    - [Validating network conflict](#validating-network-conflict)
      - [Creating an ingress network with a different range](#creating-an-ingress-network-with-a-different-range)
        - [Listing the nodes](#listing-the-nodes)
        - [Changing node availability in swarm](#changing-node-availability-in-swarm)
        - [Removing current network ingress](#removing-current-network-ingress)
        - [Creating new ingress network](#creating-new-ingress-network)
        - [Activating node availability again](#activating-node-availability-again)
    - [Creating a network for our environment](#creating-a-network-for-our-environment)
    - [Inspect all docker networks again](#inspect-all-docker-networks-again)
  - [Adding other manager nodes to the cluster](#adding-other-manager-nodes-to-the-cluster)
    - [Getting Token for manager](#getting-token-for-manager)
    - [Creating firewall rule on all nodes](#creating-firewall-rule-on-all-nodes)

## Timezone

### Check the timezone

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

### Set timezone

```bash
timedatectl set-timezone America/Sao_Paulo
```
## Tuning S.O

### Check updates

```bash
dnf clean all
dnf check-update
```

### Install utilities

```bash
dnf install -y net-tools vim nano epel-release wget curl tcpdump telnet
```

## Installing Docker

### Installing the docker repository

```bash
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

### Removing old repositories

```bash
dnf clean all
```

### Installing device mapper

```bash
dnf install -y device-mapper-persistent-data
```

### Installing the latest version of Docker

`If you need the latest version of the docker, you must manually install the latest version of containerd.io`

```bash
dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
dnf install -y docker-ce
```

### Initializing the docker daemon

```bash
systemctl enable --now docker
systemctl status docker
```

### Checking docker version

```bash
docker version
```

### Enable routing in containers

```bash
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --reload
```

### Ativando modo swarm no host

```bash
docker swarm init --advertise-addr 10.0.0.64
```

### Inspect all docker networks

```bash
for net in `docker network ls |grep -v NAME | awk '{print $2}'`;do ipam=`docker network inspect $net --format {{.IPAM}}` && echo $net - $ipam; done
```

### Validating network conflict

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

#### Creating an ingress network with a different range

##### Listing the nodes

```bash
docker node ls
```

##### Changing node availability in swarm

`AT THIS TIME IT WILL NOT START NEW SERVICES`

`<NODE_NAME> IT'S THE NAME OF THE HOSTNAME THAT RETURNS ON THE COMMAND docker node ls`

```bash
docker node update --availability drain <NODE_NAME>
```

##### Removing current network ingress

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

##### Creating new ingress network

```bash
docker network create \
--driver overlay \
--ingress \
--subnet=192.168.102.0/28 \
--gateway=192.168.102.2 \
--opt com.docker.network.driver.mtu=1200 \
ingress
```

##### Activating node availability again

```bash
docker node update --availability active <NODE_NAME>
```

### Creating a network for our environment

```bash
docker network create --driver overlay monitoring-network
```

### Inspect all docker networks again

```bash
for net in `docker network ls |grep -v NAME | awk '{print $2}'`;do ipam=`docker network inspect $net --format {{.IPAM}}` && echo $net - $ipam; done
```

## Adding other manager nodes to the cluster

### Getting Token for manager

```bash
docker swarm join-token manager
```

### Creating firewall rule on all nodes

```bash
firewall-cmd --permanent --add-port=2377/tcp
firewall-cmd --permanent --add-port=7946/tcp
firewall-cmd --permanent --add-port=7946/udp
firewall-cmd --permanent --add-port=4789/udp
firewall-cmd --reload
```