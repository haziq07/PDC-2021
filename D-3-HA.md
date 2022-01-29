<div id="top"></div>



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">Best-README-Template</h3>

  <p align="center">
    An awesome README template to jumpstart your projects!
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template">View Demo</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">Report Bug</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">Request Feature</a>
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project
Our project is about HA clusters we made this project to access a simple website.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

We build our HA clusters with the help of these below CRMs:

* https://clusterlabs.org/
* http://corosync.github.io/corosync/
* https://www.nginx.com/

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

First we setup our hosts so that we can communicate with them easily when clusters are created.

### Prerequisites

Below are the steps one by one how we configured our cluster
* npm
  ```sh
  vi /etc/hosts
  ```

### Installation

1. We will install Nginx
   ```sh
   yum install epel-release -y
   yum install nginx -y
   ```
2. Then we will set default index.html on each server
   ```sh
   ##Run below Command on HA_node_1
   echo '<h1>HA_Node_1 - osradar.com</h1>' > /usr/share/nginx/html/index.html

   ##Run below Command on HA_node_2
   echo '<h1>HA_Node_2 - osradar.com</h1>' > /usr/share/nginx/html/index.html
   ```
3. After installation we will configure pacemaker, corosync and pcsd and enable it and start it.
   ```sh
   yum install corosync pacemaker pcs -y

   systemctl enable pacemaker
   systemctl enable corosync
   systemctl enable pcsd

   systemctl start pcsd
   ```
4. Then we setup our password
   ```sh
   passwd hacluster
   ```
5. Then we create and configure our cluster.
   ```sh
   pcs host auth HA_Node_1 HA_Node_2 Master_node
   ```
6. We will start the clusters and enable them then check the status of our clusters.
   ```sh
   pcs cluster start --all
   pcs cluster enable --all
   pcs status cluster
   ```
7. Disable the STONITH and ignore the Quorum policy and check to make sure it is disabled.
   ```sh
   pcs property set stonith-enabled=false
   pcs property set no-quorum-policy=ignore
   pcs property list
   ```
8. Now we add floating IP addresses.
   ```sh
   pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=192.168.130.222 cidr_netmask=32 op monitor interval=30s

   pcs resource create webserver ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf op monitor timeout="5s" interval="5s"

   pcs status resources
   ```
9. Now we add constraint rules to our cluster for web configuration and     check our status.
   ```sh
   pcs constraint colocation add webserver virtual_ip INFINITY

   pcs constraint order virtual_ip then webserver

   pcs cluster stop --all
   pcs cluster start --all

   pcs status resources
   ```
10. Now we configure our firewall and reload it.
    ```sh
    firewall-cmd --add-service=high-availability --permanent 
    firewall-cmd --add-service=http --permanent  
    firewall-cmd --add-service=https --permanent

    firewall-cmd --reload
    ```
11. Final step is testing.
    ```sh
    pcs status nodes
    corosync-cmapctl | grep members
    pcs status corosync

    pcs cluster stop HA_Node_2 --force
    ```
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage
This project was used to access the website with virtual IP address.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [ ] Multi-language Support
    - [ ] English

<p align="right">(<a href="#top">back to top</a>)</p>


<!-- CONTACT -->
## Contact

```sh
Muhammad Rafay Nadeem 1812238 - cs1812238@szabist.pk
Ammar Ahmed 1812221 - cs1812221@szabist.pk
Muhammad Hasan Qasim 1812237 - cs1812237@szabist.pk
Mudassir Ahmed Siddiqui 1812236 - cs1812236@szabist.pk
```
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* https://searchdisasterrecovery.techtarget.com/definition/high-availability-cluster-HA-cluster
* https://www.osradar.com/create-nginx-high-availability-with-pacemaker-and-corosync-on-centos-7/
<p align="right">(<a href="#top">back to top</a>)</p>