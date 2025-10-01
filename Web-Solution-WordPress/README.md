# 📌 Web Solution with WordPress on Red Hat

This project documents the full process of deploying a WordPress Web Solution on Amazon Web Services (AWS) using Red Hat Enterprise Linux. To keep the architecture both secure and scalable, I separated the web server from the database server, allowing WordPress to connect remotely to its backend database.

The process gave me hands-on practice with:
- Configuring and managing disks and volumes using LVM (Logical Volume Manager).
- Deploying a LAMP stack (Linux, Apache, MySQL/MariaDB, PHP) on Red Hat.
- Installing and configuring WordPress to integrate with a remote database server.
- Applying security best practices, such as restricting database access to only the web server’s private IP.
---

## **Step 1: Prepare the Web Server**

1. Launch a **Red Hat** EC2 instance that will serve as a **Web Server (WordPress)**.
- Configure Security Group to allow:
    - Port 22 → SSH access for management
    - Port 80 → HTTP web traffic
<img width="1878" height="780" alt="WordPress Server 1" src="https://github.com/user-attachments/assets/b325b0f9-e167-46ce-9af9-f243a5e16c18" />

<img width="1560" height="325" alt="Red Hat Instance" src="https://github.com/user-attachments/assets/a99abfe3-0dcc-4bfd-a9c7-6367dc877ba0" />


2. Create 3 EBS volumes (10GB each) in the same Availability Zone as your instance.
    
    **Direction**:
    -** At the EC2 menu bar, under Elastic Block Storage, click on → Volume → Create Volume.
    - Volume type: General Purpose SSD (gp3)
    - Size (GiB): 10
    - Availability zone: eu-north-1b
<img width="1419" height="325" alt="Volumes 1" src="https://github.com/user-attachments/assets/45484c3d-443f-49a1-9bb3-5b3d0d8599f9" />

- Attach all 3 volumes one by one to your Web Server EC2 instance.
<img width="1872" height="684" alt="Attached Volumes" src="https://github.com/user-attachments/assets/e932267f-4fa3-4808-9f81-d9166a098ae7" />

<img width="1856" height="720" alt="Attached Volume 2" src="https://github.com/user-attachments/assets/368f3583-f811-4152-b4ff-d2180cfbaafe" />


3. Log in to the Linux terminal using 'ec2-user' (since it is a Red Hat instance):
```bash
ssh -i <Your-private-key.pem> ec2-user@<EC2-Public-IP-address>

lsblk
# Inspect the block devices that are attached to the server.
```
<img width="724" height="247" alt="image (5)" src="https://github.com/user-attachments/assets/5a122955-a05c-42dc-a028-83c5b02cb16a" />
**NOTE**: The image above shows that the 3 attached volumes are nvme0n1, nvme1n1, and nvme2n1.

- To check the current disk usage, mounted partitions, and free space on the server, run:
```bash
df -h
```
- Partition each of the disks using 'fdisk':
```bash
sudo fdisk /dev/nvme0n1
#For disk 1
```
At the `Command (m for help):` prompt, type the following:
a. **g** → to create a new GPT partition table
b. **n** → add a new partition
    - Press **Enter** to accept defaults for **partition number, first sector, and last sector**.
c. **w** → write the changes to disk and exit.
<img width="1110" height="663" alt="Fdisk" src="https://github.com/user-attachments/assets/3581539b-126c-4a5b-be92-b6eb3c63f9ab" />

- Repeat the same for the remaining disk:
```bash
sudo fdisk /dev/nvme1n1
#For disk 2

sudo fdisk /dev/nvme2n1
#For disk 3
```
- Verify the new partitions:
```bash
lsblk
```
<img width="919" height="379" alt="After partitioning" src="https://github.com/user-attachments/assets/0c1ec318-49d5-4abf-9026-33755844a7d3" />
**NOTE**: The new partitions can be easily identified by the p1 attached to each of them: nvme0n1p1, nvme1n1p1 and nvme2n1p1


4. Install LVM2 (Logical Volume Manager) to manage the 3 volumes as one pool and scan for disks/partitions available for LVM:
```bash
sudo yum install lvm2

sudo lvmdiskscan
```
<img width="877" height="276" alt="Newly Partitioned disk" src="https://github.com/user-attachments/assets/d66e6430-9d1a-4a72-9e43-8524cf239bc9" />

- Use 'pvcreate' to create 3 physical volumes for the partitions:
```bash
sudo pvcreate /dev/nvme0n1p1
# First

sudo pvcreate /dev/nvme1n1p1
# Second

sudo pvcreate /dev/nvme2n1p1
# Third
```

- To verify that the Physical volume has been created, run:
```bash
sudo pvs
```
<img width="636" height="138" alt="image (6)" src="https://github.com/user-attachments/assets/a6ad8418-d9ba-4693-b56c-7adc1179d60b" />


5. Add all 3 PVs to a volume group (VG). Let’s name the group 'webdata-vg':
```bash
sudo vgcreate webdata-vg /dev/nvme0n1p1 /dev/nvme1n1p1 /dev/nvme2n1p1
```

- To confirm that the VG has been created, run:
```bash
sudo vgs
```
<img width="683" height="80" alt="image (7)" src="https://github.com/user-attachments/assets/a43993c4-f479-47be-a819-f0904c6c82ca" />


6. 1. Use `lvcreate` to create 2 logical volumes.
Note:
- **apps-lv → 14G** (for website files under /var/www/html).
- **logs-lv → 14G** (for logs under /var/log)

```bash
sudo lvcreate -n apps-lv -L 14G webdata-vg

sudo lvcreate -n logs-lv -L 14G webdata-vg

sudo lvs
# Verify the logical volume has been created
```
<img width="1129" height="246" alt="logical volumes created" src="https://github.com/user-attachments/assets/53377f51-379d-4763-9c8f-d08c445b5d0d" />


7. Verify the entire setup:
```bash
sudo vgdisplay -v
# Verifies that the VG, PVs, and LVs are correctly set up

sudo lsblk
```
<img width="849" height="437" alt="image (8)" src="https://github.com/user-attachments/assets/04ebb645-9853-4fb8-b84a-2141e5ec99cc" />


8. Use 'mkfs.ext4' to format the logical volumes:
```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv

sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```


9. Create directories to mount the volumes:
```bash
sudo mkdir -p /var/www/html

sudo mkdir -p /home/recovery/logs
```

- Mount apps-lv to Apache’s web root:
```bash
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

- Use rsync to back up existing logs and then mount logs-lv to the logs directory:
```bash
sudo rsync -av /var/log/ /home/recovery/logs/

sudo mount /dev/webdata-vg/logs-lv /var/log
```

- Now restore the backed-up log files:
```bash
sudo rsync -av /home/recovery/logs/ /var/log
```


10. Get UUIDs of mounted partitions for permanent mount setup:
```bash
sudo blkid
```
<img width="1770" height="348" alt="Log and App" src="https://github.com/user-attachments/assets/47d837e6-7c4a-4ad8-8ff4-0a5f2ecfe623" />

- We need these mounts to persist after reboots, so let's update the /etc/fstab file:
```bash
sudo vi /etc/fstab
```

- Add UUIDs:
```bash
UUID=3d6418d2-2a5d-4809-8a93-834a4ad828d9   /var/www/html       ext4   defaults   0 0
UUID=9158c7e4-7fdb-4799-b1b3-c2a2ae74722f   /home/recovery/logs ext4   defaults   0 0
```
<img width="1318" height="145" alt="uuid" src="https://github.com/user-attachments/assets/78031d6e-7070-4615-a4e2-fe54f159db08" />


11. Test the configuration and reload the daemon:
```bash
sudo mount -a

sudo systemctl daemon-reload

df -h
# Verify your setup
```
<img width="1248" height="535" alt="Log App" src="https://github.com/user-attachments/assets/e777fa25-be6d-4130-b31e-4b9ef22eba49" />

---

## **Step 2: Prepare the Database Server**

1. Launch a second **Red Hat** EC2 instance that will serve as a DB Server.
- Configure Security Group to allow Port 22 (TCP): SSH access for management.
<img width="1551" height="90" alt="image (9)" src="https://github.com/user-attachments/assets/b523a7b5-f330-4f34-bbb8-7469c8afe711" />

- Create three EBS volumes (each 10GB) in the same Availability Zone as your instance.
- Attach all 3 volumes one by one to your DB Server EC2 instance.


2. Log in to the Linux terminal using 'ec2-user':
```bash
ssh -i <Your-private-key.pem> ec2-user@<EC2-Public-IP-address>
```
- Check your block devices:
```bash
lsblk

df -h
```
<img width="615" height="236" alt="image (10)" src="https://github.com/user-attachments/assets/386125e3-3ece-42e4-bbb7-81f46b825891" />
**NOTE**: The image above shows that the 3 attached volumes are nvme1n1, nvme2n1, and nvme3n1.

- Partition each of the disks using fdisk :
```bash
sudo fdisk /dev/nvme1n1
#For disk 1
```
At the `Command (m for help): prompt, type the following:
a. **g** → to create a new GPT partition table
b. **n** → add a new partition
    - Press **Enter** to accept defaults for partition number, first sector, and last sector.
c. **w** → write the changes to disk and exit.

- Repeat the same for the remaining disk
```bash
sudo fdisk /dev/nvme2n1
#For disk 2

sudo fdisk /dev/nvme3n1
#For disk 3
```
<img width="1095" height="652" alt="DB Disk" src="https://github.com/user-attachments/assets/839e2b18-ac97-4409-bf21-c50cc0c3b9f8" />

- Verify the new partitions:
```bash
lsblk
```
<img width="792" height="364" alt="DB Disk 1" src="https://github.com/user-attachments/assets/322d8d90-4c55-445d-b939-27ca2fdd20ee" />


3. Install LVM2 (Logical Volume Manager) to manage the 3 volumes as one pool and scan for disks/partitions available for LVM:
```bash
sudo yum install lvm2

sudo lvmdiskscan
```
<img width="661" height="253" alt="LVM" src="https://github.com/user-attachments/assets/d804d078-e76c-48a4-bf97-ee7b9bc50048" />

- Use 'pvcreate' to create 3 physical volumes for the partitions:
```bash
sudo pvcreate /dev/nvme1n1p1
# First

sudo pvcreate /dev/nvme2n1p1
# Second

sudo pvcreate /dev/nvme3n1p1
# Third
```
- To verify that the Physical volume has been created, run:
```bash
sudo pvs
```
<img width="586" height="136" alt="image (11)" src="https://github.com/user-attachments/assets/183c999b-5fea-47b0-b45a-63bf97edd447" />


4. Add all 3 PVs to a volume group (VG). Let’s name the group 'webdata-vg':
```bash
sudo vgcreate database-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1

sudo vgs
# Confirm that the VG has been created
```
<img width="699" height="78" alt="image (12)" src="https://github.com/user-attachments/assets/7c1c39a6-8663-4158-bd0d-be888fc07f62" />


5. Use lvcreate to create a single LV for the database:
```bash
sudo lvcreate -n db-lv -L 14G database-vg

sudo lvs
# Verify the logical volume has been created
```
<img width="1049" height="86" alt="image (13)" src="https://github.com/user-attachments/assets/7a9275dc-e87f-462c-a4d5-20885de45623" />


6. Verify the entire setup:
```bash
sudo vgdisplay -v
# Verifies that the VG, PVs, and LVs are correctly set up

sudo lsblk
```
<img width="972" height="403" alt="vgdisplay" src="https://github.com/user-attachments/assets/b3809fab-373f-40a4-bf19-45fa7b7538a6" />


7. Use 'mkfs.ext4' to format the logical volumes:
```bash
sudo mkfs.ext4 /dev/database-vg/db-lv
# Create an ext4 filesystem on the LV

sudo mkdir -p /db
# Create the mount point directory

sudo mount /dev/database-vg/db-lv /db
# Mount the filesystem
```


8. Gets UUIDs of mounted partitions for permanent mount setup:
```bash
sudo blkid

sudo vi /etc/fstab
```
<img width="1780" height="279" alt="DB 9" src="https://github.com/user-attachments/assets/56058144-c7d4-41b5-abcb-f0b8bda99adb" />

- Add UUIDs:
```bash
UUID=3419c338-dde5-4c1f-bd47-9e59d2c20f9d   /db   ext4   defaults   0 0
```
<img width="1263" height="180" alt="DB 10" src="https://github.com/user-attachments/assets/3fca8e6d-6074-4fef-90a0-d4427580ef6d" />


9. Test the configuration and reload the daemon:
```bash
sudo mount -a

sudo systemctl daemon-reload

df -h
#Verify your setup
```
<img width="1189" height="447" alt="DB 11" src="https://github.com/user-attachments/assets/f913f531-8fd3-4b5a-b403-8500925b90da" />

---

## **Step 3: Install WordPress on the Web Server**

1. Update the repository, install wget, Apache, and its dependencies:
```bash
sudo yum -y update

sudo yum install wget httpd php-fpm php-json
```

- Install PHP (We'll use the Remi repository for this):
```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

- Install additional PHP extensions required by WordPress:
```bash
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
```

- Verify the PHP Version using 'php -v':
<img width="925" height="186" alt="PHP version" src="https://github.com/user-attachments/assets/c687acc8-ad67-4a4c-994f-43f1f2cf7346" />

- Start and enable PHP-FPM:
```bash
sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo systemctl status php-fpm
```
<img width="1743" height="346" alt="image (14)" src="https://github.com/user-attachments/assets/7a68de43-d1a5-4624-ba86-642e5fb44fcd" />

- Allow Apache Execution:
```bash
sudo setsebool -P httpd_execmem 1

sudo systemctl restart httpd
```

2. Test PHP:
```bash
sudo vi /var/www/html/info.php
# Create a test PHP file

<?php
phpinfo();
?>
# Add this configuration
```
- Open your browser and visit: 'http://<Web-Server-Public-IP>/info.php'
<img width="1486" height="740" alt="image (15)" src="https://github.com/user-attachments/assets/81716201-aeea-4330-98e0-744baaf55ea0" />


3. Download and set up WordPress:
```bash
sudo mkdir wordpress && cd wordpress

sudo wget http://wordpress.org/latest.tar.gz

sudo tar xzvf latest.tar.gz

cd wordpress/

sudo cp -R wp-config-sample.php wp-config.php

cd ..

sudo cp -R wordpress/. /var/www/html/
```

4. Configure SELinux policies:
```bash
sudo chown -R apache:apache /var/www/html

sudo chcon -t httpd_sys_rw_content_t /var/www/html -R

sudo setsebool -P httpd_execmem 1

sudo setsebool -P httpd_can_network_connect=1

sudo setsebool -P httpd_can_network_connect_db=1
```

- Restart Apache:
```bash
sudo systemctl restart httpd
```

---


## **Step 4: Install MySQL (MariaDB) on the DB Server**
```bash
sudo yum update -y

sudo dnf install -y mariadb-server

sudo systemctl enable mariadb

sudo systemctl start mariadb

sudo systemctl status mariadb
```
<img width="1368" height="472" alt="Running msql" src="https://github.com/user-attachments/assets/0bfdc689-4ff0-40a1-aca3-9f02ea2e63c0" />


## **Step 5: Configure Database for WordPress**

1. Set a root password and tighten security:
```bash
sudo mysql -u root -p
```
<img width="985" height="302" alt="image (16)" src="https://github.com/user-attachments/assets/42e69de3-322b-42e1-87b9-8a5140c6bee0" />


2. Create a database, a user, and grant permissions for the Web Server Private IP:
```bash
CREATE DATABASE wordpress_db;

CREATE USER `name_user`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'PasswordHere';

GRANT ALL ON wordpress_db.* TO 'name_user'@'<Web-Server-Private-IP-Address>';

FLUSH PRIVILEGES;

SHOW DATABASES;

exit
```
<img width="586" height="388" alt="Wordpress DB" src="https://github.com/user-attachments/assets/98be16d2-34c8-47c9-99eb-19bb358c73e3" />

---

## **Step 6: Connect WordPress (web server) to the Remote Database**

1. Open MySQL/Aurora (port 3306) on the DB server EC2.
For extra security, use the Web Server’s private IP address (for secure DB access).
<img width="1552" height="202" alt="Inbound rule" src="https://github.com/user-attachments/assets/52e98897-5f6a-4e17-a53a-8ac0f8106bf4" />


2. Install MYSQL client:
```bash
sudo dnf install -y mariadb

mysql -h <DB-Server-Private-IP> -u username -p

SHOW DATABASES;
```
<img width="987" height="463" alt="Database" src="https://github.com/user-attachments/assets/8af5d43e-1606-42cc-a70a-2f33c686d0d8" />


3. Update WordPress configuration:
```bash
cd /var/www/html/

sudo vi wp-config.php
```

- Set:
```bash
define( 'DB_NAME', 'database_name_here' );
define( 'DB_USER', 'username_here' );
define( 'DB_PASSWORD', 'password_here' );
define( 'DB_HOST', 'localhost' );
# Localhost is the DB Server Private IP
```
<img width="967" height="391" alt="image (17)" src="https://github.com/user-attachments/assets/4d52aa4f-b98d-4aef-9129-d02549733d87" />

---

## **Step 7: Access WordPress**

- Access WordPress from your browser using: 'http://<Web-Server-Public-IP-Address>/'
<img width="1236" height="946" alt="image (18)" src="https://github.com/user-attachments/assets/0719d7c0-e991-499d-bc0d-950b3968b9b3" />

<img width="1140" height="383" alt="image (19)" src="https://github.com/user-attachments/assets/9620abc2-fdd9-4d92-b50d-c05c5d1c3794" />

<img width="1896" height="887" alt="wordpress done" src="https://github.com/user-attachments/assets/40b758a9-2a21-44dd-8a29-49716ab74ca3" />

*We have successfully set up WordPress on AWS using separate Web and DB servers. Your WordPress site is now ready for content creation and customization.*


---



# **🔧 Troubleshooting**

## ❌ Issue 1: EPEL 8 Package Error
When trying to install PHP dependencies with EPEL 8 on my web server, I encountered an error when I used:
```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```
<img width="1521" height="382" alt="Troubleshooting 1" src="https://github.com/user-attachments/assets/19503519-cef8-46e3-8dd5-cd2b928c3a0a" />


✔️ **To Fix**:
- I verified my OS version using:
```bash
cat /etc/redhat-release
```

**The Result**:
Red Hat Enterprise Linux 10

- So I used the EPEL 9 package instead:
```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```


## ❌ Issue 2: MySQL Server Package Not Found

Attempting to run 'sudo yum install mysql-server' failed on my Database server because RHEL 9 does not provide a package called 'mysql-server'.
<img width="1098" height="261" alt="troubleshooting 2" src="https://github.com/user-attachments/assets/60844ece-13d6-4770-91d9-23f33231243d" />


✔️ **Fix**:
- I installed MariaDB instead, which is fully compatible with WordPress.
```bash
sudo dnf install -y mariadb-server

sudo systemctl enable mariadb

sudo systemctl start mariadb

sudo systemctl status mariadb
```
<img width="1368" height="472" alt="Running msql (1)" src="https://github.com/user-attachments/assets/ecce009e-cd0f-4f78-b554-4bf3658d3cec" />


