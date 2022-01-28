
# HPC

In this project we are configuring High Performance
Computer which is used for resource extensive 
computation. When resources of Single Computers are 
not enough for performing compute intesive task we use
different computer to perfrom Parallel execution.



## Configuration

For disabling Common UNIX Printing System

```bash
  chkconfig –level 35 cups off
```

For adding IP’s of of all virtual machines in hosts file
```bash
  vi /etc/hosts
    192.168.95.136		MasterNode
    192.168.95.135		Node1
    192.168.95.129		Node2
```
Copying this hosts file to Node1 and Node2
```bash
  scp /etc/hosts Node1:/etc/
  scp /etc/hosts Node2:/etc/
```
Generating private and public key for root user in master node
```bash
  ssh-keygen –t dsa
  ssh-keygen –t rsa
```
Adding all keys/.pub files in single files
```bash
  cat *.pub >> authorized_keys
```
Copying all the files of .ssh directory including .pub,authorized_keys,know_hosts to everynode
```bash
  scp –r .ssh Node1:/root/
  scp –r .ssh Node2:/root/
```
Generate nodes ssh fingerprints
```bash
  ssh-keyscan –t dsa MasterNode Node1 Node2
  ssh-keyscan –t rsa MasterNode Node1 Node2
```
Adding these generated keys in known_hosts
```bash
  ssh-keyscan –t dsa MasterNode Node1 Node2 > /etc/ssh/ssh_known_hosts
  ssh-keyscan –t rsa MasterNode Node1 Node2 >> /etc/ssh/ssh_known_hosts
```
Copying this known_hosts file to every node
```bash
  scp /etc/ssh/ssh_known_hosts Node1:/etc/ssh/
  scp /etc/ssh/ssh_known_hosts Node2:/etc/ssh/
```
Installing chrony on Master and ComputeNodes
```bash
  yum install chrony
```
Configuring NTP service via chrony
```bash
  systemctl start chronyd
  systemctl enable chronyd
```
Commenting pool for stop syncing it with Internet 
```bash
  Pool 2.centos.pool.ntp.org iburstpool
```
Editing chrony.conf file to synchronize it with local time
```bash
  vi /etc/chrony.conf
    server 192.168.95.136
    allow 192.168.95.0/24
    local stratum 10
```
Restart chrony to reflect changes
```bash
  systemctl restart chronyd
```
Editing chrony.conf file of Node1,Node2 to synchronize it with MasterNode time
```bash
  vi /etc/chrony.conf
    server 192.168.95.136
```
Updating yum database to get pdsh in yum list
```bash
  dnf makecache –refresh
```
Installing pdsh using yum
```bash
  dnf -y install pdsh
```
list down machines name in files
```bash
  vi /etc/machines
    MasterNode
    Node1
    Node2
```
Installing NFS
```bash
  dnf install nfs-utils
```
Configuring NFS
```bash
  systemctl start nfs-server.service
  systemctl enable nfs-server.service
  systemctl status nfs-server.service
```
Defining folder for mounting in NFS
```bash
  vi /etc/exports
    /cluster		*(rw,no_root_squash,sync)
```
Start NFS service
```bash
  chkconfig –level 35 nfs-server on
```
Creating directory for mounting
```bash
  mkdir /cluster
```
Creating directory on Node1 and Node2 both
```bash
  pdsh –w “ssh:root@Node[1-2]” mkdir /cluster
```
Mounting /cluster directory in Node1 and Node2
```bash
  mount –t nfs MasterNode:/cluster /cluster/
```
Make entry in fstab to we shouldn’t have to mount it again after reboot
Do this for Node1 and Node2 both
```bash
  vi /etc/fstab
    MasterNode:/cluster	/cluster		nfs	defaults	0 0
```
Adding group & user on All nodes (MasterNode, Node1, Node2)
```bash
  groupadd –g 600 mpigroup
  useadd –u 600 –g 600 /cluster/mpiuser mpiuser
```
Adding Packages
```bash
  dnf install gcc		(compiler of C,C++,…)
  dnf install gcc-fortran	(fortan support)
``` 
Creating public and private keys for mpiuser
```bash
  ssh-keygen –t dsa
  ssh-keygen –t rsa
```
Adding this key to authorized keys
```bash
  cat *.pub >> authorized_keys
```
Changing ownership of /cluster directory
```bash
  chown mpiuser:mpigroup /cluster -R
```
Downloading MPICH package
```bash
  wget https://www.mpich.org/static/downloads/4.0/mpich-4.0.tar.gz
```
Untar this package
```bash
  tar xzf mpich-4.0.tar.gz
```
Configure MPICH package
```bash
  ./configure –prefix=/cluster/mpich2
```
Compiling all the options that were detected at time of configuration
```bash
  make
```
Placing this complied files into /cluster/mpich2
```bash
  make install
```
Modifying bash profile of mpiuser
```bash
  vi .bash_profile
    PATH=$PATH:$HOME/bin:/cluster/mpich2/bin
    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/cluster/mpich2/lib
    export PATH
    export LD_LIBRARY_PATH
```
Load new values of bash_profile
```bash
  source .bash_profile
```
Adding computer nodes name in mpiuser/mpd.hosts
```bash
  vi /cluster/mpiuser/mpd.hosts
    Node1
    Node2
```
Adding secret word in .mpd.conf file
```bash
  vi /cluster/mpiuser/.mpd.conf
    secret_word=redhat
```