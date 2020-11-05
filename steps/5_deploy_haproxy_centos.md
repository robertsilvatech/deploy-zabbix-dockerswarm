# Instalação HA-PROXY no Centos 8

## HA-PROXY

https://upcloud.com/community/tutorials/haproxy-load-balancer-centos/
https://medium.com/@warlord77/haproxy-2-0-centos-7-setup-6ca655bec840


dnf install gcc pcre-devel tar make -y

cd /tmp

wget http://www.haproxy.org/download/2.0/src/haproxy-2.0.14.tar.gz -O haproxy.tar.gz

tar xzvf haproxy.tar.gz -C /tmp

cd haproxy-2.0.14/

make TARGET=linux-glibc

sudo make install

```
sudo mkdir -p /etc/haproxy
sudo mkdir -p /var/lib/haproxy 
sudo touch /var/lib/haproxy/stats
sudo ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy
sudo cp examples/haproxy.init /etc/init.d/haproxy
sudo chmod 755 /etc/init.d/haproxy
sudo systemctl daemon-reload
sudo chkconfig haproxy on
sudo useradd -r haproxy
haproxy -v
```

cd /etc/haproxy/
vim haproxy.cfg

haproxy -f haproxy.cfg

- Create firewall rule
```
firewall-cmd --permanent --add-port=80/tcp
<<<<<<< HEAD
firewall-cmd --permanent --add-port=10051/tcp
firewall-cmd --reload
```

- Look for the HAPROXY process and kill it 
=======

firewall-cmd --permanent --add-port=3000/tcp. 
firewall-cmd --permanent --add-port=8080/tcp. 
firewall-cmd --permanent --add-port=8081/tcp. 
firewall-cmd --permanent --add-port=8082/tcp. 
firewall-cmd --permanent --add-port=10051/tcp. 
firewall-cmd --reload. 
>>>>>>> 40ee159e7e56a3572a957c656b9dd2291ead66f5

```
ps aux |grep haproxy
kill -9 PID_HAPROXY
```


- Start the HAPROXY service and check the status

```
systemctl start haproxy
systemctl status haproxy
<<<<<<< HEAD
```
=======
>>>>>>> 40ee159e7e56a3572a957c656b9dd2291ead66f5
