
# GROUP MEMEBERS NAMES:

KAMRAN YAQOOB - 1812187, AQSA SHABIR - 1812180 , MUSKAN MOMIN - 1812196 , ASHISH ALI UMATIA 1812181

# CONFIGURE AND MAINTAIN HIGH AVAILABILTY CLUSTER:
HA Cluster is simply refers to a quality of a system to operate continuously without failure for a long period of time.

# INTRODUCTION:
We have implement a HA Cluster for only one node provides the service with the secondary node(s) taking over if it fails and a cluster is made up of two or more computers commonly known as nodes that work together to perform a task.

We have setup a cluster and also we need at least two servers.
1)node1.ha.com: 192.168.229.135
2) node2.ha.com: 192.168.229.134


## Configure Local DNS Setting

We open and edit a file

```bash
  vim /etc/hosts
```
## Install Nginx 
We have install nginx web server

```bash
  yum install epel-release
  yum install nginx
  systemctl enable nginx
  systemctl start nginx
  echo "default page for node1.ha.com" | sudo tee /usr/share/nginx/html/index.html
default page for node1.ha.com
echo "default page for node2.ha.com" | sudo tee /usr/share/nginx/html/index.html
default page for node1.ha.com
```
## Install Corosync and Pacemaker
We have to install Pacemaker, Corosync, and Pcs on each node

```bash
  yum install corosync pacemaker pcs
  systemctl enable pcsd
  systemctl start pcsd
```
## Create a Cluster
The system user called “hacluster” is created and the password is hacluster123

```bash
   passwd hacluster
   pcs cluster auth node1.ha.com node2.ha.com -u hacluster -p hacluster123 --force
   pcs cluster setup --name newcluster node1.ha.com node2.ha.com --force
   pcs cluster enable --all
   pcs cluster start --all
   pcs status
```
## Configure Cluster Options:
```bash
   pcs property set stonith-enabled=false
   pcs property set no-quorum-policy=ignore
   pcs property list
```
## Add Cluster Resource/ Services:
```bash
   pcs resource create floating_ip ocf:heartbeat:IPaddr2 ip=192.168.229.136 cidr_netmask=24 op monitor interval=60s
   pcs resource create http_servers ocf:heartbeat:nginx configfile="/etc/nginx/nginx.conf" op monitor timeout="205" interval="60s"
   pcs status resources 
```


