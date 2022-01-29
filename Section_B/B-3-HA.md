# **High Availability Cluster with DRBD, Pacemaker and Heartbeat**
We will be making HA cluster with DRDB, Pacemaker and Corosync/heartbeat. Synchronization will happen between two nodes will allow the replication of the data.

## Changing the hostname of node 1:

    hostname node1

## Changing the hostname of node :

    hostname node2

## Checking the ip of both node1 and node2:

    ip a

## Adding both ips in hosts file:


    vim /etc/hosts
 
 Adding the ip addresses:

     192.168.139.144 node1
     192.168.139.145 node2

## Installing ntp services in both nodes:

    yum install ntp


# Opening ntp server configuration and after configuration restarting the service in both nodes:

    vim /etc/ntp.conf
    service ntpd restart
   
   Checking if the servers works:
   

    watch ntpq -p -n

# Partitioning on Node 1 using fdisk for cluster servers and creating LVM Partition on both nodes:
Checking the disks details:

    fdisk -l
 Going into the sdb disk:

    fdisk /dev/sdb

Now we press n for creating new partition. Then p for making primary. Then press enter by selecting everything default. To convert the type of the disk we press t and then 8e which is to convert to Linux LVM. To save the changes file we press w.

Creating physical volume:

    pvcreate /dev/sdb1

Creating volume group:

    vgcreate vgdrbd /dev/sdb1
Creating logical volume:

    lvcreate -n lvdrbd /dev/mapper/vgdrbd -L 4000M+

# DRBD Setup

Installing drbd and kmod in both nodes:

    yum -y install drbd84-utilis kmod-drbd84

Configuring the drbd file on both nodes:

    vim /etc/drbd.conf

```
resource r0 {
protocol C;
on node1 {
device /dev/drbd0;
disk /dev/vgdrbd/lvdrbd;
address 192.168.139.144:7788;
meta-disk internal;
}
on node2 {
device /dev/drbd0;
disk /dev/vgdrbd/lvdrbd;
address 192.168.139.145:7788;
meta-disk internal;
}
}
```
Using scp command to copy node 1 configuration file to node 2:

    scp /etc/drbd.conf node2:/etc/drbd.conf

Creating metadata block in both nodes:

    drbdadm crate-md r0

Disabling SELINUX Because we don’t need this and we are not using in production service:

    vim /etc/sysconfig/selinux

```
SELINUX=disabled
```
We must reboot the system after disabling the SELINUX.

Making node 1 primary and starting resource r0:

    drbadm up r0
    drbdadm -- primary all

Checking the status of drbd:

    drbdadm status

Mounting drbd0:

   mkfs.ext4 /dev/drbd0

# Pacemaker and Corosync/Heartbeat Setup

Installing pacemaker, corosync and pcs both nodes:

    yum -y install corosync pacemaker pcs

Enabling the services of pcs, pacemaker and corosync in both nodes:

    system enable pcsd
    system enable corosync
    system enable pacemaker
    systemctl start pcsd

Authorizing the servers with hacluster and pcs commands in node 1:

    pcs cluster auth node1 node2
We must enter the username and the password must be the password of your machine.

Setting up the cluster and defining the name of the servers. Starting cluster services in both nodes:

    pcs cluster start --all

Checking the status of pcs and corosync:

    pcs status
    
Checking the status of corosync:

    pcs status corosync
**Now our corosync, pcs, pacemaker and drbd has started working and can communicate with each other.**
