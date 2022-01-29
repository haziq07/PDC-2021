<div id="top"></div>



<!-- PROJECT LOGO -->
<br />
<div align="center">
  
  </a>

  <h3 align="center">HA Cluster Implementation Using AWS</h3>

  <p align="center">
    "Parallel & Distributed Computing Project"
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template"><strong></strong></a>
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
In this project we have utilised HAProxy which is a very fast and reliable solution for high availability, load balancing, It supports TCP and HTTP-based applications. Nowadays most of the websites need 99.999% uptime for their site, which is not possible with single server setup. Then we need some high availability environment that can easily manage with single server failure.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With
We have utlised AWS for creating VM Instances & Putty inorder to access & configure those VM's.

* [AWS](https://aws.amazon.com/)
* [Putty](https://www.putty.org/)


<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

Inorder to implement HA Cluster we need atleast one HA Proxy Server with minimum two VM's for App Server.

### Prerequisites

- AWS Account


### Installation

_For HAProxy_

1. Installing HA Proxy
   ```sh
   sudo yum install haproxy
   ```
2. Configuring HA Proxy
   ```sh
    /etc/haproxy/haproxy.cfg
   ```
3. Start HA Proxy
   ```js
   systemctl start haproxy
   ```
   ```js
   systemctl enable haproxy
   ```

_For App Server_

1. Installing Apache HTTP Server
   ```sh
    yum install httpd
   ```
2. Starting/Enabling Apache HTTP Server
   ```sh
    systemctl start httpd
   ```
   ```sh
    systemctl enable httpd
   ```
3. Configuring Apache HTTP Server
   ```sh
    vim /usr/share/httpd/npindex/index.html
   ```
   ```sh
    yum install vim -y
   ```
   ```sh
    vim /usr/share/httpd/noindex/index.html
   ```

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage
<img src= HACluster.jpeg>
High availability clusters are used for mission important applications including databases, eCommerce websites, and transaction processing systems. High availability clusters are often used for load balancing, backup, and failover purposes.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [ ]Installing haproxy
- [ ]Install apache
- [ ]Configured the loadbalancing(ha proxy)
- [ ]Configured the apache
- [ ]Connecting the apache server to ha proxy server
- [ ]Deploy the sample webpage to apache



<p align="right">(<a href="#top">back to top</a>)</p>


<!-- LICENSE -->
## License

There is no need for a license all the software used are open sourced however a $1 Activation Fee is charged for activating AWS account on trial period wihich lasts 12 Months with some limitations.
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

- Ahsan Hamid         - cs1812218@szabist.pk
- Bushra Malik Shaikh - cs1812224@szabist.pk
- Sana Shujrah        - cs1812245@szabist.pk
- Sunila Zulfiqar     - cs1812248@szabist.pk



<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

Use this space to list resources you find helpful and would like to give credit to. I've included a few of my favorites to kick things off!


* [Inatlling & Configuring Apache Server](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7)
* [Inatlling & Configuring HAProxy](https://cloudwafer.com/blog/installing-haproxy-on-centos-7/)


<p align="right">(<a href="#top">back to top</a>)</p>





<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/othneildrew/Best-README-Template/issues
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.png