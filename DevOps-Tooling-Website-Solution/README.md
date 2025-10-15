# 🚀 DevOps Tooling Website Solution

## Overview
This project implements a DevOps Tooling Website Solution designed to make access to essential DevOps tools within a corporate infrastructure easily accessible and highly available. The setup demonstrates how **multiple web servers** share application data and connect to a **single database** using a **Network File System (NFS)** for shared storage, following an enterprise-style three-tier architecture:
- Presentation Layer: Apache Web Server hosting the Tooling Website
- Application Layer: PHP-based WordPress-like application
- Database Layer: MySQL database hosted on a separate server

This setup mimics a real-world DevOps environment that prioritizes scalability, fault tolerance, and centralized data management.


## 🎯 Project Objective

The objective of this project is to design and implement a highly available, scalable, and centralized web solution that mirrors real-world DevOps infrastructure. Specifically, the goals are to:
- Deploy a Tooling Website that provides access to DevOps tools through a web interface.
- Configure a three-tier architecture using distinct Web, Database, and NFS Storage servers.
- Enable multiple web servers to share the same application files using NFS (Network File System) for synchronization and consistency.
- Implement a centralized database using MySQL that can be securely accessed by all web servers.
- Utilize Logical Volume Management (LVM) to create scalable, flexible storage partitions.
- Apply hands-on Linux administration, networking, and security practices in a cloud environment.


## 🧩 Architecture Diagram and Concept Explanation

### Infrastructure Components:
- Platform: AWS EC2 Instances
- Operating Systems:
    - Web Servers → Red Hat Enterprise Linux (RHEL 9)
    - Database Server → Ubuntu 24.04 LTS
    - Storage Server → RHEL 9 (NFS)
- Programming Language: PHP
- Database: MySQL
- Storage: NFS (shared between web servers)
- Code Repository: GitHub

### Architecture Pattern:
Multiple stateless web servers share:
- A common database (MySQL)
- A shared filesystem via NFS
This allows all servers to access identical web content and logs while maintaining a centralized data source, which is a foundational concept in high-availability web architecture.

---

## Step 1 - Prepare the NFS Server

1. Launch an EC2 instance with RHEL 9 (Red Hat Enterprise Linux) Operating System.
- Configure Security Group to allow:
    - Port 22 → SSH access for management


2. Create three EBS volumes (each 10GB) in the same Availability Zone as your instance, and attach each volume one at a time to your RHEL 9 EC2 instance.


3. Log in to the Linux terminal using `ec2-user` (since it is a Red Hat instance):
```bash
ssh -i <Your-private-key.pem> ec2-user@<EC2-Public-IP-address>
```


- Inspect the block devices that are attached to the server:
```bash
lsblk
```


*This image shows that the Root Disk is 'nvme0n1' , and the 3 attached volumes are `nvme1n1, nvme2n1 and nvme3n1`*


- Partition each of the disks using `fdisk`:

**NOTE**: At the `Command (m for help):` prompt, type the following:

a. **g** → to create a new GPT partition table
b. **n** → add a new partition
    - Press **Enter** to accept defaults for **partition number, first sector, and last sector**.
c. **w** → write the changes to disk and exit.

```bash
sudo fdisk /dev/nvme1n1
#For disk 1

sudo fdisk /dev/nvme2n1
#For disk 2

sudo fdisk /dev/nvme3n1
#For disk 3
```

- Verify the new partitions:
```bash
lsblk
```


*The new partitions can be easily identified by the p1 attached to each of them: `nvme1n1p1, nvme2n1p1 and nvme3n1p1`*


4. Install LVM2 (Logical Volume Manager) to manage the 3 volumes as one pool and scan for disks/partitions available for LVM:
```bash
sudo yum install lvm2

sudo lvmdiskscan
```


- Use `pvcreate` to create 3 physical volumes for the partitions:
```bash
sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1 

sudo pvs
# Verify that the Physical volume has been created
```


5. Add all 3 PVs to a volume group (VG). Let’s name the group `nfs-vg` :
```bash
sudo vgcreate nfs-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1

sudo vgs
# Confirm that the VG has been created
```

- - Use `lvcreate` to create 3 logical volumes:

**Note:**

a. **apps-lv → 9G** (for website files under `/var/www/html`).
b. **logs-lv → 9G** (for logs under `/var/log`)
c. **opt-lv → 9G** (for optional software and third-party applications under `/opt`)
```bash
sudo lvcreate -n apps-lv -L 9G nfs-vg

sudo lvcreate -n logs-lv -L 9G nfs-vg

sudo lvcreate -n opt-lv -L 9G nfs-vg
```
```bash
sudo lvs
# Verify the logical volume has been created
```


- Use `mkfs.xfs` to format the logical volumes.
```bash
sudo mkfs.xfs /dev/nfs-vg/apps-lv

sudo mkfs.xfs /dev/nfs-vg/logs-lv

sudo mkfs.xfs /dev/nfs-vg/opt-lv
```

- Create mount points on the `/mnt` directory
```bash
sudo mkdir -p /mnt/apps /mnt/logs /mnt/opt

sudo mount /dev/nfs-vg/apps-lv /mnt/apps

sudo mount /dev/nfs-vg/logs-lv /mnt/logs

sudo mount /dev/nfs-vg/opt-lv  /mnt/opt
```


6. Install NFS server, then configure it to start on reboot:
```bash
sudo yum update -y

sudo yum install nfs-utils -y

sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service
```


7. Export the Mounts for the webservers' `Subnet CIDR` to connect as Clients.
- Set up permissions that will allow the Web Servers to read, write, and execute files on NFS:
```bash
sudo chown -R nobody: /mnt/apps /mnt/logs /mnt/opt

sudo chmod -R 777 /mnt/apps /mnt/logs /mnt/opt

sudo systemctl restart nfs-server.service
```


- Configure access to NFS for clients within the same subnet.
    
    **DIRECTION**
    - To check your subnet CIDR: Open your EC2 details in the AWS web console, and locate the '**Networking**' tab and open the Subnet link. Mine is `10.0.0.0/24`


```bash
sudo vi /etc/exports
```

- Edit (my **Subnet-CIDR** is `10.0.0.0/24`):
```bash
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```


- Apply export rules and allow web servers to mount directories:
```bash
sudo exportfs -arv

sudo systemctl restart nfs-server

sudo exportfs -v
```


- Add new inbound rules. For the NFS Server to be accessible from the client, set the following inbound rule by opening ports: TCP 111, UDP 111, UDP 2049, and TCP 2049. Also, set the web server `Subnet CIDR` as the source in the security group. 


---


## Step 2 - Configure the Database Server

1. Launch a second **EC2 instance (Ubuntu)** that will serve as a DB Server.
- In the Security Group:
    - Allow **SSH (port 22)** for your IP (for management).
    - Allow **MySQL/Aurora (port 3306)** and use the **Subnet CIDR** as the source (for secure DB access).


- SSH into the Ubuntu EC2 instance:
```bash
ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

- Install and upgrade Ubuntu:
```bash
sudo apt update && sudo apt upgrade -y

sudo apt install mysql-server -y

sudo mysql_secure_installation
# Run MySQL secure installation
```

- Create a user, and grant privileges:
```sql
sudo mysql

CREATE DATABASE tooling;

CREATE USER 'webaccess'@'DB-private IP' IDENTIFIED BY 'StrongPassword!';

GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'DB-private IP';

FLUSH PRIVILEGES;

SHOW DATABASES;

EXIT;
```


2. Open MySQL configuration file:
```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```

- Find the line `bind-address = 127.0.0.1` and change it to:
```bash
bind-address = 0.0.0.0
```
*This allows remote connections. This allows MySQL to accept connections from **any IP address**, not just localhost.*


- Restart MySQL to apply changes:
```bash
sudo systemctl restart mysql

sudo systemctl status mysql
```

---


## Step 3 - Prepare the 3 Web Servers

**In this step, we will create 3 different web servers all in the same subnet, and carry out this exact step in each of them.**

We need to ensure that the Web Servers can serve the same content from the shared storage solutions (NFS Server and MySQL database).

We will do the following:
- Configure NFS client (must be done on all three servers)
- Deploy a Tooling application to our Web Servers into a shared NFS folder
- Configure the Web Servers to work with a single MySQL database


1. Launch an EC2 instance with RHEL 9 (Red Hat Enterprise Linux) Operating System.
- Configure Security Group **(which)** to allow**:**
    - Port 22 → SSH access for management
    - Port 80 → HTTP from 0.0.0.0/0
    - Port 443 → HTTPS from 0.0.0.0/0


2. SSH into the server and update the server:
```bash
ssh -i <Your-private-key.pem> ec2-user@<EC2-Public-IP-address>

sudo dnf update -y
```

3. Install Apache and PHP-FPM, then enable and start services :
```bash
sudo dnf install -y httpd php php-fpm php-mysqlnd php-json php-xml php-gd php-cli php-common php-opcache nfs-utils

sudo systemctl enable --now httpd

sudo systemctl enable --now php-fpm
```

- Set SELinux:
```bash
sudo setsebool -P httpd_execmem 1

sudo setsebool -P httpd_can_network_connect 1

sudo setsebool -P httpd_can_network_connect_db 1
```

4. Mount  `/var/www/` and target the NFS server's export for apps:
```bash
sudo mkdir -p /var/www

sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

sudo vi /etc/fstab
```

- Add the following line:
```bash
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```

- Confirm mounts:
```bash
df -h
```

**Now, proceed to create the remaining two servers and repeat this step for each of them.**


---

## Step 4 - Verify Apache Files and Directories

Verify that Apache files and directories are available on the **Web Servers** in `/var/www` and also on the **NFS** Server in `/mnt/apps`. If the same files are present in both locations, it means NFS was mounted correctly.

1. Create a test file on **Web Server 1** and verify its presence on **Web Server 2 & 3** :
```bash
# Run both commands on web server 1:
sudo touch /var/www/test.txt
ls /var/www

# Run this command on web server 2 & 3:
ls /var/www

# Run this command on the NFS Server:
ls /mnt/apps
```

2. Mount the Apache log folder `(/var/log/httpd)` on your web servers to the NFS server’s log export `(/mnt/logs)`:
```bash
sudo mkdir -p /var/log/httpd

sudo mount -t nfs -o rw,nosuid <NFS-Private-IP>:/mnt/logs /var/log/httpd
```

- To make sure the changes persist after reboot:
```bash
sudo vi /etc/fstab
```

- Add the following line:
```bash
<NFS-Private-IP>:/mnt/logs /var/log/httpd nfs defaults 0 0
```

- Reload and verify:
```bash
sudo systemctl daemon-reload

df -h
```


---

## Step 5 - Test MySQL Remote Connection

Choose any of the **web servers** and test to confirm remote MySQL connectivity:

- Install MySQL client on the 3 web servers:
```bash
sudo dnf install -y mariadb

mysql -h <DB-Private-IP> -u webaccess -p -e "SHOW DATABASES;"
```


---

## Step 6 - Fork the Tooling Source Code

1. Go to the StegHub GitHub account and fork the tooling source code:


2. Deploy the Tooling Website's Code to the 3 Web Server:
```bash
sudo yum install git -y

git clone https://github.com/Trivancee/tooling.git

# Copy the tooling/html to /var/www/html directory:
sudo cp -R ~/tooling/html/* /var/www/html/
```

3. Adjust Permissions:
```bash
sudo chown -R apache:apache /var/www/html

sudo chmod -R 755 /var/www/html

sudo setenforce 0
```

- Change the line:
```bash
sudo vi /etc/sysconfig/selinux
```

- Once inside the editor, look for this line: `SELINUX=enforcing`, then change it to `SELINUX=disabled`:


- Restart:
```bash
sudo systemctl restart httpd
```
***Remember to carry out this step on all the 3 web servers***


---

## Step 7 - Update the website's database configuration

- Run this on each of the web servers:
```bash
sudo vi /var/www/html/functions.php
```

- Replace it with your own details:
```bash
$db = mysqli_connect('DB_Private-IP', 'DB_Username', 'DB_Password', 'tooling');
```

- Apply the tooling-db.sql script:
```bash
sudo mysql -h <databse-private-ip> -u <db-username> -p tooling < ~/tooling/tooling-db.sql
```

---

## Step 8 - Create a New Admin User in MySQL

- Run this on the Database Server:
```bash
sudo mysql -u <db-username> -p -h <db-Private-IP>

USE tooling;

INSERT INTO users (id, username, password, email, user_type, status)

VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');

EXIT;
```

- Open your browser and go to `http://<Web-Server-Public-IP>/index.php` using your web server 1, 2 and 3 IP addresses :




At this point, our project has been successfully completed. The LAMP stack web solution is fully deployed with proper separation of concerns across the Web, NFS, and Database tiers. All services are running, integrated, and accessible, demonstrating an enterprise-grade deployment approach.



## 🧰 Troubleshooting

**PROBLEM: Database Server Rejected Connection from Web Server** 


While setting up the connection between the Web Server and the Database Server, the web application failed to connect to MySQL. The error message indicated that the MySQL user webaccess was not allowed to connect from the host (the web server’s private IP)

**CAUSE:** By default, MySQL restricts user access based on both the username and the host from which the connection originates.
In my case, the user `webaccess` was created with permissions limited to my Database private IP address, meaning MySQL was rejecting any connections coming from a different machine (the web servers)

**FIX**: 
To allow the web servers to connect to the database securely, I modified the MySQL user’s host permissions to include the web server’s private subnet.

```bash
sudo mysql -u root -p

CREATE USER 'webaccess'@'10.0.0.%' IDENTIFIED BY 'your_password1';

GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'10.0.0.%' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```
This allows any instance within the subnet `10.0.0.x` to connect using the `webaccess` user.


## Security Implications

For this project where I have 3 different web servers, using 10.0.0.% is perfectly fine and practical as I’m only limiting access to my private subnet (e.g., 10.0.0.0/24). So it’s secure using 10.0.0.% because:
- The **security group** on my DB server only allows inbound connections from `10.0.0.0/24` subnet and not from the public internet.
- The **user I created has a strong password**.

This way, even though MySQL would *accept* connections from anywhere inside the private network, no external host can reach the server at all.