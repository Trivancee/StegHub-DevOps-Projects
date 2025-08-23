#  Deploying a LAMP stack website in AWS Cloud

This project demonstrates how to deploy a **LAMP stack** (Linux, Apache, MySQL, PHP) on an AWS EC2 Ubuntu 24.04 instance.  

## 📌 Introduction

**LAMP Stack** is a set of open-source technologies commonly used together to build and deploy dynamic websites and web applications. It consists of four main components:
- **Linux** → Operating system that hosts the environment  
- **Apache** → Web server that handles HTTP requests  
- **MySQL** → Database system that stores application data  
- **PHP** → Scripting language for dynamic content  

These components work together to deliver dynamic web applications.

##  Project Prerequisites

- AWS EC2 instance: Ubuntu 24.04 LTS (t3.micro – Free Tier)  
- Security group rules:
  - **Port 22 (SSH):** For secure access  
  - **Port 80 (HTTP):** To serve web traffic  
- SSH key pair for authentication  
- GitHub for a central repository for version control and project documentation 
- Visual Studio Code (VS Code): The primary code editor and SSH client integration 
- A browser to test web responses  

---


## 1. Launch an EC2 Instance

1. Log into your AWS Management Console and navigate to EC2 → Launch Instance.
2. Set the following configurations:
- Instance Name: My LAMP Stack Server
- AMI: Ubuntu Server 24.04 LTS (64-bit x86/ARM)
- Instance Type: t3.micro (Free Tier eligible)
3. Create or select an existing key pair:
- If creating a new key pair, give it a name eg,: StegHubserver-keypair (this key will be saved in your downloads)
4. Configure networking:
- Attach the instance to the desired VPC/Subnet.
- Enable a Security Group allowing inbound traffic for:
-  **SSH (port 22)** from your IP.
-  **HTTP (port 80)** from anywhere.
5. Launch the instance. 
<img width="766" height="142" alt="Ubuntu Server" src="https://github.com/user-attachments/assets/55fa513d-d974-4423-8ea1-4d78102e878d" />


---

## 2. Connect to the Instance

```bash
cd Downloads

ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
#Replace <private-key-name> with your key name and replace <Public-IP-address> with your instance's public IP address. 
```
<img width="1421" height="763" alt="image (1)" src="https://github.com/user-attachments/assets/8a517406-a13b-4ea5-a96c-8bae58786d13" />

---

## 3. Install Apache

```bash
sudo apt update

sudo apt install apache2 -y

sudo systemctl status apache2
# Checks the status of the Apache service to confirm it is running.
```
<img width="1412" height="491" alt="image (2)" src="https://github.com/user-attachments/assets/eda4ec7c-31ef-4b26-bb21-8d5f0ac1ef20" />


#### ✅ Verify by visiting `http://<Public-IP>:80` in your browser.
<img width="924" height="420" alt="apache2 default page" src="https://github.com/user-attachments/assets/1fe5cebb-34f7-4a7a-a51f-6172a0972b07" />


---

## 4. Install MySQL

```bash
sudo apt install mysql-server

sudo mysql
```
<img width="688" height="203" alt="mysql" src="https://github.com/user-attachments/assets/381bf3bb-2f47-4569-ab92-017facc23bef" />


Set a password and secure MySQL:


```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; 

mysql> exit
```

```bash
sudo mysql_secure_installation
```

---

## 5. Install PHP

```bash
sudo apt install php libapache2-mod-php php-mysql

php -v
# Verifies PHP installation.
```
<img width="478" height="146" alt="PHP installed" src="https://github.com/user-attachments/assets/770cb574-0ffc-4e9c-9d86-2edea1b95471" />


---

## 6. Configure Apache Virtual Host

Create project directory:
```bash
sudo mkdir /var/www/projectlamp

sudo chown -R $USER:$USER /var/www/projectlamp
```

Create virtual host config:
```bash
sudo vi /etc/apache2/sites-available/lampstack.conf
```

Paste the following:

```bash
<VirtualHost *:80>
    ServerName lampstack
    ServerAlias www.lampstack
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    <Directory /var/www/projectlamp>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable new site and disable default:
```bash
sudo a2ensite lampstack
sudo a2dissite 000-default
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Create a Test Page:
```bash
sudo echo 'Hello LAMP from hostname' $(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X-aws-ec2-metadata-token: $TOKEN" \
-s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' \
$(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X-aws-ec2-metadata-token: $TOKEN" \
-s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
This creates an index.html that dynamically shows the instance’s hostname and IP.

Check in browser:
```bash
http://<Public-IP>:80
# Replace Public-IP with your public IP address
```

<img width="563" height="125" alt="Hello LAMP" src="https://github.com/user-attachments/assets/668954b7-1e3a-44e6-b773-f4338f561472" />

---

## 7. Enable PHP on the website

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

Add the following:
```bash
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
Save and exit.

Reload Apache:
```bash
sudo systemctl reload apache2
```

Create PHP info page:
```bash
vim /var/www/projectlamp/index.php
```

Paste:
```bash
<?php
phpinfo();
?>
```
Save and exit.

Next, refresh the web page in your browser. Expected result:
<img width="1490" height="293" alt="image" src="https://github.com/user-attachments/assets/8e77bd57-7ebb-4eb8-8d1d-9ee771592976" />

---

## 8. Final Steps 

Remove info.php after testing for security purposes:
```bash
sudo rm /var/www/projectlamp/index.php
```

At this point, your LAMP stack is fully installed and ready for hosting web applications.








