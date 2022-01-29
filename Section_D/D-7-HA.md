# High Availability DRBD Cluster File System Pacemaker

High availability is one of the important quality attributes of a cloudbased application or service. High availability of all the system components such as the application itself, storage, database etc. contributes to the high availability of the entire application. Since most of the cloudbased applications and services, in general, are dynamic databasedriven webbased applications, database plays an important role in the system`s high availability. Recently, cluster-based multimaster  database replication techniques such  as MariaDB Galera Cluster and MySQL Cluster have become available as a more effective alternatives.

## Features

A highly available architecture prevents this situation using the following process:
- Detecting failure
- Performing failover of the application to another redundant host
- Restarting or repairing the failed server without requiring manual intervention

## Tech

- The heartbeat technique
- DRBD - Setup Linux Disk Replication 
- NTP with Chrony
- Pacemaker and High Availability RPMS

## Installation and Configuration
All the nodes of the high availability clusters must be able to communicate each other using FQDN so here is the output of /etc/hosts from all the cluster nodes
```sh
[root@Node1 ~]# cat /etc/hosts
```
![alt text](https://i.ibb.co/kmhHgjW/img.jpg)
```sh
[root@Node1 ~]# vim /etc/hosts
```
```sh
[root@Node1 ~]# scp /etc/hosts 192.168.119.134 
```

Instaling Chrony 

The sources subcommand is also useful because it provides information about the time source configured in chrony.conf.
```sh
[root@Node2 ~]# chronic sources
```
![alt text](https://i.ibb.co/g3gv967/eaba65eb-1fa1-4355-bd4c-1d64f1a63f6f.jpg)

Chronic tracking

The chronyc command, when used with the tracking subcommand, provides statistics that report how far off the local system is from the reference server.
![alt text](https://i.ibb.co/6NjctPD/78b5ab2f-7beb-4bc2-aade-19d6f17be8f2.jpg)

So before we install CentOS HA Cluster rpms, we will enable the HighAvailability repo
```sh
[root@Node1 ~]# dnf config-manager --set-enabled HighAvailability
```
List the enabled repos
```sh
[root@Node1 ~]# dnf repolist
```
Install pacemaker Linux and other high availability rpms using DNF which is the default package manager in Node2.
```sh
[root@Node1 ~]# dnf install pcs pacemaker fence-agents-all -y
```
corosync-cfgtool is a tool for displaying and configuring active parameters within corosync
```sh
[root@Node1 ~]# corosync-cfgtool -s
```
![alt text](https://i.ibb.co/Rhn9S3n/06ed17f3-b020-414e-999d-51655842f180.jpg)

Check the status of corosync across cluster nodes
```sh
[root@Node1 ~]# pcs status corosync
```
![alt text](https://i.ibb.co/cCwWcwr/cb8b7832-6284-496f-b2eb-31833944c12f.jpg)
```sh
[root@Node1 ~]# cat /etc/chrony.conf
```
![alt text](https://i.ibb.co/K5Gd956/da191230-40c2-4014-8a8c-049ab2e9e323.jpg)

Lower level device for DRBD Configuration

Perform below steps on only any one of the cluster nodes
```sh
[root@Node2 ~]# lsblk 
```
Create physical volume on /dev/sdb
```sh
[root@Node2 ~]# pvcreate /dev/sdb 
```
You can also verify the mounted partitions on Node2
```sh
[root@Node2 ~]# vgs  
```
```sh
[root@Node2 ~]# vg extend Node2 /dev/sdb 
```
Create logical volume cluster_lv
```sh
[root@Node2 ~]# lvcreate -L 4G -n drbd-1 Node2 
```
![alt text](https://i.ibb.co/NFwfLWs/f299bdc1-6d41-486e-8be1-d17b7a6badb5.jpg)
```sh
[root@repo-server ~]# rpm --https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```
```sh
[root@repo-server ~]# rpm -Uvh https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```
```sh
[root@Node2 ~]# dnf search drbd 
```
```sh
[root@Node2 ~]# dnf install kmod-drbd90.x86_64 drbd90-utils.x86_64 -y
```
```sh
[root@Node2 ~]# cat /etc/drbd.conf
```

We will use the auto-promote feature of DRBD, so that DRBD automatically sets itself Primary when needed, this will probably apply to all of your resources. 
```sh
[root@Node2 ~]# cat /etc/drbd.d/global_common.conf
```
![alt text](https://i.ibb.co/1K7cMts/a7a670e0-9442-4eb9-a94a-f269623fa849.jpg)
![alt text](https://i.ibb.co/yFwsPRc/637ee30f-56b3-4244-88ea-991bf0df4633.jpg)

Next copy this file manually across all the cluster nodes
```sh
[root@Node2 ~]# scp /etc/drbd.d/global_common.conf Node2;/etc/drbd.d/
```
```sh
[root@Node2 ~]# cat /etc/drbd.d/drbd1.res
```
![alt text](https://i.ibb.co/WP7WJjR/f80e5ae0-ab9f-4fe4-82d6-3f0a172fe119.jpg)
```sh
[root@Node1 ~]# scp /etc/drbd.d/drbd1.res Node2:/etc/drbd.d/
root@Node2's password:
```

Create mount points for your HA Logical Volumes on all the cluster nodes
```sh
[root@Node2 ~]# mkdir /var/lib/drbd
```
```sh
[root@Node2 ~]# drbdadm create-md drbd1
```
```sh
[root@Node2 ~]#  firewall-cmd --add-port 7789/tcp --permanent
```
```sh
[root@Node2 ~]#  firewall-cmd --reload
```
```sh
[root@Node2 ~]#  drbdadm up drbd1
```
Next check the DRBD resource status
```sh
[root@Node2 ~]#  drbdadm status drbd1
```
![alt text](https://i.ibb.co/zxVjxSW/20952a8b-bfda-48e4-89b0-1fcf290cd3a5.jpg)
```sh
[root@Node2 ~]#  drbdadm primary --force drbd1
```
Next check the DRBD resource status
```sh
[root@Node2 ~]#  drbdadm status drbd1
```
```sh
[root@Node2 ~]#   ls -l /dev/drbd/by-disk/Node2/drdb-1
```
```sh
[root@Node2 ~]#   ls -l /dev/drbd/by-disk/Node2/drdb-1
```
```sh
[root@Node2 ~]#   mkfs.ext4 /dev/drbd1
```
```sh
[root@Node2 ~]#   mkdir /share
```
```sh
[root@Node2 ~]#   mount /dev/drbd1 /share
```
The DRBD device is also mounted on Node2 cluster node
```sh
[root@Node2 ~]#   df -h /share/
```
Next check the DRBD resource status
```sh
[root@Node2 ~]#  drbdadm status drbd1
```
Using DRBD as a background service in a pacemaker cluster
```sh
[root@Node2 ~]#cat /etc/drbd.d/global_common.conf
```
```sh
[root@Node1 ~]# scp /etc/drbd.d/global_common.conf
Node2:/etc/drbd.d/
```
