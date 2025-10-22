# Implementing a Load Balancer Solution With Apache

In production environments, users enter a single URL, but behind the scenes, traffic is distributed across multiple servers. This approach, known as **load balancing**, ensures scalability, availability, and fault tolerance. When website traffic grows, we can **scale vertically** (add more CPU/RAM to one server) or **scale horizontally** (add more servers). Horizontal scaling is more flexible, as it enables systems to handle large or fluctuating loads with ease.

This project introduces a **Load Balancer (LB)** to the existing DevOps Tooling Website solution. The load balancer acts as a single entry point that distributes traffic evenly across multiple web servers. This guide will walk you through the process of deploying and configuring an Apache Load Balancer for a Tooling Website solution.


## Prerequisites

Before we begin, make sure you have the following:
- Two RHEL 8 Web Servers
- One MySQL Database Server (Ubuntu 24.04)
- One RHEL 8 NFS Server


## Step 1: Configure Apache As a Load Balancer

1. Launch an Ubuntu EC2 instance and name it **Project-8-apache-lb**
- Configure Security Group to allow:
    - Port 22 → SSH access for management
    - Port 80 → HTTP from 0.0.0.0/0

<img width="1596" height="270" alt="All servers" src="https://github.com/user-attachments/assets/93feb9cd-435a-4881-b378-83976cbd2fcd" />

- SSH into the EC2 instance:
```bash
#ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

2. Install Apache and its dependencies
```bash
sudo apt update && sudo apt upgrade -y

sudo apt install apache2 -y

sudo apt-get install libxml2-de
```

- Enable the following Modules:
```bash
sudo a2enmod rewrite

sudo a2enmod proxy

sudo a2enmod proxy_balancer

sudo a2enmod proxy_http

sudo a2enmod headers

sudo a2enmod lbmethod_bytraffic
```

- Restart apache2 service:
```bash
sudo systemctl restart apache2

sudo systemctl status apache2
```
<img width="562" height="228" alt="Enabled" src="https://github.com/user-attachments/assets/de60c66d-1f33-4cf8-95ba-cc34816374fb" />


3. Configure load balancing:
```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

- Add this configuration into this section `<VirtualHost *:80>  </VirtualHost>`
```bash
<Proxy "balancer://mycluster">
    BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
    BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
    ProxySet lbmethod=bytraffic
    # ProxySet lbmethod=byrequests
</Proxy>

ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```
<img width="1525" height="780" alt="Virtual host configuration" src="https://github.com/user-attachments/assets/ad87040c-8a1c-4a50-b422-71d66bcf8dff" />

<img width="1278" height="778" alt="After configuration" src="https://github.com/user-attachments/assets/948e8cc4-c3a9-4f86-a84d-cffb7c44e0f6" />

*This configuration uses the by-traffic balancing method, which distributes incoming load between the Web Servers based on current traffic load. The loadfactor parameter controls the proportion of traffic distribution.*


- Restart the  Apache server:
```bash
sudo systemctl restart apache2
```

4. Test load balancer via web browser using the LB's public IP address: 
```bash
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
```
<img width="1440" height="762" alt="Load balancer IP" src="https://github.com/user-attachments/assets/1b5dc2e7-0b60-4843-a553-1c12bcbfa086" />


5. Open two SSH consoles, for the 2 Web Servers, and run Unmount `/var/log/httpd/` from both (if they were mounted previously):
```bash
df -h

sudo systemctl stop httpd

sudo umount /var/log/httpd

sudo systemctl restart httpd

sudo systemctl status httpd
```

- On each Web Server, run:
```bash
sudo tail -f /var/log/httpd/access_log
```
<img width="1780" height="838" alt="log files" src="https://github.com/user-attachments/assets/82185f65-d939-49a5-a812-5b5c43c9cc73" />

*Refresh the load balancer web page several times. You should see new records appear in each web server's log files (as seen in the screenshot above). The number of requests to each server should be approximately the same since we set the load factor to the same value for both servers.*



## Step 2: Configure IP address to Domain Name Mapping
1. Open this file on your Load Balancer server:
```bash
sudo vi /etc/hosts
```

- Add these records into the file to replace the IP addresses with the arbitrary names for both servers:
```bash
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
<img width="1014" height="442" alt="Web 1 and 2" src="https://github.com/user-attachments/assets/5655aa5e-d7b7-4ba7-996a-67c5d96bf38f" />


2. Update the Load Balancer config file with these names:
```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

- Replace the IP addresses with the arbitrary names:
```bash
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
<img width="838" height="319" alt="image (8)" src="https://github.com/user-attachments/assets/f317638e-d28a-436d-aec4-113edf89e618" />


3. Test the configuration by curling the Web Servers from the Load Balancer:
```bash
curl http://Web1

curl http://Web2
```

<img width="1554" height="726" alt="Curl http web" src="https://github.com/user-attachments/assets/989c83d7-60f8-44f2-a832-c544cf2f721b" />

**Congratulations. You have just implemented a load-balancing Web Solution for your DevOps team.**
