## CONFIGURE APACHE AS A LOAD BALANCER

In this project we will enhance our Tooling Website solution in Project 7 [Link](https://github.com/adakolito/darey.io-pbl/blob/main/Project7.md)  by adding a Load Balancer to distribute traffic between Web Servers and allow users to access our website using a single URL.

A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

Perequistes: Ensure to have these server configured

1. Two RHEL8 Web Servers

2. One MySQL DB Server (based on Ubuntu 20.04)

3. One RHEL8 NFS server

# Step 1

Created all the instances as shown:

![Ec2-Instance 1](https://user-images.githubusercontent.com/10111342/130513683-7563e056-b77b-4af3-9a7d-4afeeea414bd.png)

# Step 1

Open TCP port 80 on Project-8-apache-l by creating an Inbound Rule in Security Group.

I have a SG set to all traffic in my instance. This is for practice purposes only.

# Step 3

Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers

```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```
**Make Apache is running**

`sudo systemctl status apache2`

![Apache Installed and running 2](https://user-images.githubusercontent.com/10111342/130513803-41975da5-e205-40e4-8b56-747be9c0edf9.png)


Configure load balancing on the load balancer instance

```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2
```

![Configure load balancer 3](https://user-images.githubusercontent.com/10111342/130515150-22c0ea86-58ad-4b94-9108-c9dd1efcd3ad.png)




Verified that the webservers has its own log directory




`sudo tail -f /var/log/httpd/access_log`


![Access to the web servers 5](https://user-images.githubusercontent.com/10111342/130516044-5d40520c-9a69-4159-b848-c8b9968eb6b4.png)


## Step 4
Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php


![Load Balancer working 4](https://user-images.githubusercontent.com/10111342/130516367-2502814a-5f66-40e4-9223-abe99461e62e.png)




**Optional Step – Configure Local DNS Names Resolution**

```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

![DNS Load balancer config 6](https://user-images.githubusercontent.com/10111342/130516737-dc665d99-7425-4513-87a3-3ea972b0b35c.png)

*Curl web1 and it I can see the html file from the local host

![Curl web1 7](https://user-images.githubusercontent.com/10111342/130517065-7ff204a3-a030-4ea3-87f9-bf41dae706e8.png)


*Project Summary Diagram*

![Diagram of Project summary 8](https://user-images.githubusercontent.com/10111342/130517367-ce119ec7-51b7-42cc-a039-8f0fbaef857c.png)