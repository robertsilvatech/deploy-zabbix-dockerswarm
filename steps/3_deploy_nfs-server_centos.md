# Instalação NFS-Server no Centos

- [Instalação NFS-Server no Centos](#instalação-nfs-server-no-centos)
  - [Timezone](#timezone)
    - [Check the timezone](#check-the-timezone)
    - [Set timezone](#set-timezone)
  - [Tuning S.O](#tuning-so)
    - [Check updates](#check-updates)
    - [Install utilities](#install-utilities)
  - [NFS-Server](#nfs-server)
    - [Create directory for data to be shared between docker nodes](#create-directory-for-data-to-be-shared-between-docker-nodes)
    - [Install NFS](#install-nfs)
    - [Configuring exports](#configuring-exports)
    - [Initializing the service](#initializing-the-service)
    - [Releasing Firewall](#releasing-firewall)
  - [NFS-Client on CentOS](#nfs-client-on-centos)
    - [Install utilities on the client](#install-utilities-on-the-client)
    - [Install NFS on the client](#install-nfs-on-the-client)
    - [Create directory for the mount point](#create-directory-for-the-mount-point)
    - [Validating the mount point](#validating-the-mount-point)
    - [Check if it worked](#check-if-it-worked)
    - [Removendo ponto de montagem temporario](#removendo-ponto-de-montagem-temporario)
    - [Configuring permanent mount point](#configuring-permanent-mount-point)
    - [Force the fstab reload](#force-the-fstab-reload)

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

## NFS-Server

### Create directory for data to be shared between docker nodes

```bash
mkdir -p /data/data-docker
```

### Install NFS

```bash
dnf install nfs-utils -y
```

### Configuring exports

```shell
vim /etc/exports
/data/data-docker/ *(rw,sync,no_root_squash,no_subtree_check)
exportfs -ar
```

### Initializing the service

```bash
systemctl enable --now nfs-server
systemctl status nfs-server
```

### Releasing Firewall

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

## NFS-Client on CentOS

`REMEMBER TO CHECK THE DATE AND TIME OF THE SERVERS`

### Install utilities on the client

```bash
dnf install -y net-tools vim nano epel-release wget curl tcpdump
```

### Install NFS on the client

```bash
dnf install nfs-utils -y
```

### Create directory for the mount point

```shell
mkdir -p /mnt/data-docker
```

### Validating the mount point

```bash
mount -v -t nfs 10.0.0.65:/data/data-docker /mnt/data-docker
```

### Check if it worked

```bash
df -h
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         7.8G     0  7.8G   0% /dev
tmpfs                            7.8G     0  7.8G   0% /dev/shm
tmpfs                            7.8G   20M  7.8G   1% /run
tmpfs                            7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root           44G  2.2G   42G   5% /
/dev/sda1                       1014M  173M  842M  17% /boot
/dev/mapper/VG_DOCKER-LV_DOCKER   49G   53M   46G   1% /var/lib/docker
tmpfs                            1.6G     0  1.6G   0% /run/user/0
**10.0.0.60:/data/data-docker    99G   60M   94G   1% /mnt/data-docker**
```

### Removendo ponto de montagem temporario

```bash
umount /mnt/data-docker
```

### Configuring permanent mount point

```bash
vim /etc/fstab
10.0.0.65:/data/data-docker   /mnt/data-docker    nfs    defaults    0 0 
```

### Force the fstab reload

```bash
sudo mount -av
/                        : ignored
/boot                    : already mounted
swap                     : ignored
/var/lib/docker          : already mounted
/mnt/data-docker         : already mounted
```

