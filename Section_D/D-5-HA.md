su ------------------------------root-------------- nano /etc/hosts
192.168.187.133 node1 192.168.187.134 node2 ping node1 ping node2 yum
repolist all \| grep -i ha dnf repository-packages ha dnf config-manager
--set-enabled ha dnf repository-packages ha list ha repo list dnf
--enablerepo=ha -y install pacemaker pcs systemctl enable --now pcsd
passwd hacluster pcs host node1 node2 pcs cluster start --all pcs
cluster enable --all pcs status cluster pcs property set
stonith-enabled=false pcs property set no-quorum-policy=ignore pcs
property list15) pcs resource create virtual\_ip ocf:heartbeat:IPaddr2
ip=192.168.187.222 cidr\_netmask=32 op monitor interval=30s pcs resource
create webserver ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf op
monitor timeout="5s" interval="5s" pcs constraint colocation add
webserver virtual\_ip INFINITY pcs constraint order virtual\_ip then
webserver pcs cluster stop --all pcs cluster start --all pcs status
resources firewall-cmd --add-service=ha --permanent firewall-cmd
--add-service=http --permanent firewall-cmd --add-service=https
--permanent firewall-cmd --reload for reloading firewall pcs status
nodes corosync-cmapctl \| grep members pcs status corosync pcs cluster
stop node2 --force yum install -y heartbeat heartbeat-pills
heartbeat-stonith heartbeat-dlevel install -y squid
