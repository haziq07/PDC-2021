**High Availability Cluster**

**1: Linking between machines:**

First we need to check IP on both Centos machines, `HAServer1` and `HAServer2` by typing command:
```
ifconfig
```
IP's of both machines: `192.168.71.133` HAServer1 `192.168.71.134` HAServer2 Now we have to add them in the hosts file the command will be
```
vim /etc/hosts
```
Add IP address in Server Host File:
```
192.168.71.133 HAServer1
192.168.71.134 HAServer2
```
Now save the files and type the following commands on each machine to check if they are connected.
```
ping HAServer1
ping HAServer2
```
**2: Stop unnecessary services and run the IP tables command**
At this stage first we need to disable services like cups.path. The following command will be used to disable it.
```
systemctl disable cups.path
```
Now we need to check that the command is actually inactive.
```
systemctl status cups.path
```
Flush iptables firewall on both servers:
```
iptables -F
```
check output type the below command.
```
iptables -L
```
Then save both iptables

**3: Configure NTP in Both Machines:**
```
rpm -qa | grep 'chrony'
```
Now we need to enable our machine IP by changing the entry in chrony's configuration file for NTP application.
```
vim /etc/chrony.conf
```
Now, we have to allow both our machine's IPs.
```
allow 192.168.71.133
allow 192.168.71.134
```
Restart the chrony service.
```
systemctl restart chronyd
```
Now, type this command to allow the NTP incoming requests.
```
firewall-cmd --permanent --add-service=ntp
firewall-cmd --reload
```
**4:Create Partitions on Both Machines For DRDB:**
```
fdisk -l
```
The second step is to create partitions in the sdb.
```
fdisk /dev/sdb
```
Now press n for creating new partitions
```
n
```
press p for primary.
```
p
```
Now press enter by selecting all options as default. A new partition will be created.
To change the type from Linux to Linux LVM we need to change the HEX code to 8e. For this we have type t then provide code 8e. Now save all changes by pressing W.

**Creating physical volume**
```
pvcreate /dev/sdb1
```
**Creating Volume Group**  
```
vgcreate vgdrbd /dev/sdb1
```
**Creating Logical Volume**  
```
lvcreate -n lvdrbd /dev/mapper/vgdrbd -L +4000M
```
Now make the filesystem
```
mkfs.ext3 /dev/mapper/vgdrbd-lvdrbd
```
**5: Including entries in the SYSCTL config file:**
We will add few entries in the sysctl config file of both the machines
```
vi etc/sysctl.conf
```
```
net.ipv4.conf.ens33.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.ens33.arp_announce = 2
```
response:
```
sysctl -p
```
**6: INSTALLING AND CONFIGURING DRBD**
install drbd on both machines:  
`dnf -y install https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/d/drbd-9.17.0-1.el8.x86_64.rpm`

install drbd-utils:  
`dnf -y install https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/d/drbd-utils-9.17.0-1.el8.x86_64.rpm`

you can install kernel module for drbd:  
`dnf -y install https://mirror.rackspace.com/elrepo/elrepo/el8/x86_64/RPMS/kmod-drbd90-9.1.5-1.el8_5.elrepo.x86_64.rpm`

After complete drbd installation, create a new file, and make changes.
```
nano /etc/drbd.d/resource0.res

resource resource0 {
protocol C;
on HAServer1 {
device /dev/drbd0;
disk /dev/vgdrbd/lvdrbd;
address 192.168.71.133:7788;
meta-disk internal;
}
on HAServer2 {
device /dev/drbd0;
disk /dev/vgdrbd/lvdrbd;
address 192.168.71.134:7788;
meta-disk internal;
}}
```
Allow port 7788 on the firewall to avoid any problems and reload it.
`firewall-cmd --zone=public --permanent --add-port 7788/tcp`
`firewall-cmd --reload`

Start DRBD
```
modprobe drbd
```
Now echo modprobe drd in rc.local
```
echo "modprobe drbd" >> /etc/rc.local
```
Initialize the DRBD resource 
```
drbdadm create-md resource0
```
enable resource:
```
drbdadm up resource0
```
Check the status of the drbd:
```
drbdadm status resource0
```
start the initial full synchronization On your first machine:
```
drbdadm primary --force resource0
```
check the status:
```
drbdadm status resource0
```
Your output will change from the previous one.

Now create the filesystem:
```
mkfs.ext4 /dev/drbd0
```
Mount the DRBD device on any directory, we are using /opt:
```
mount /dev/drbd0 /opt/
```
Create some files inside /opt directory by using command:
```
cd /opt
touch file1 file2 file3
```
On the first machine, unmount the DRBD and make it secondary:
```
cd
umount /opt
drbdadm secondary resource0
```
On the second machine, make the second node primary with this command:
```
drbdadm primary resource0
```
mount the DRBD device on the /opt directory with the following command:
```
mount /dev/drbd0 /opt
```
print the list of files in the /opt directory by typing below command:
```
ls /opt
```
All files stored in the DRBD device will be displayed there. Here, DRBD installation and configuration is successful.

**7: INSTALLING AND CONFIGURING PACEMAKER**

enable the High Availability Repo and install pacemaker:
```
dnf --enablerepo=ha -y install pacemaker pcs
```
Now, enable pcs:
```
systemctl enable --now pcsd
```
Set the password for the cluster admin:
```
passwd hacluster
```
Allow the HA service:
```
firewall-cmd --add-service=high-availability --permanent
firewall-cmd --reload
```
create authorization among nodes and configure basic cluster settings:
```
pcs host auth HAServer1 HAServer2
pcs cluster setup ha_cluster HAServer1 HAServer2
```
Start services for cluster:

```
pcs cluster start --all
```

Make it auto-start:
```
pcs cluster enable --all
```
check status:
```
pcs cluster status
```
output
```
Cluster Status:
 Cluster Summary:
   * Stack: corosync
   * Current DC: HAServer2 (version 2.1.0-8.el8-7c3f660707) - partition with quorum
   * Last updated: Tue Jan 25 11:39:34 2022
   * Last change:  Mon Jan 24 03:34:14 2022 by hacluster via crmd on HAServer1
   * 2 nodes configured
   * 0 resource instances configured
 Node List:
   * Online: [ HAServer1 HAServer2 ]

PCSD Status:
  HAServer1: Online
  HAServer2: Online
```
check the status for corosync:
```
pcs status corosync
```
output
```
Membership information
----------------------
    Nodeid      Votes Name
         1          1 HAServer1
         2          1 HAServer2 (local)
```
**8: SETTING UP SQUID**

Install squid
```
dnf install squid -y
```
Back up the configuration file:
```
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.ori
```
Access the configuration file:
```
vim /etc/squid/squid.conf
```
Allow squid on the firewall:
```
firewall-cmd --add-service=squid --permanent 
firewall-cmd --reload
```
configuration of CentOS client:
```
sudo vim /etc/profile.d/proxyserver.sh
```
Add the following settings:
```
MY_PROXY_URL="HAServer1:3128" 
HTTP_PROXY=$MY_PROXY_URL
HTTPS_PROXY=$MY_PROXY_URL
FTP_PROXY=$MY_PROXY_URL
http_proxy=$MY_PROXY_URL
https_proxy=$MY_PROXY_URL
ftp_proxy=$MY_PROXY_URL
```
Then finally source the file:
```
source /etc/profile.d/proxyserver.sh
```
Installed Successfully
