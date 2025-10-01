# 📌 Web Solution with WordPress on Red Hat

This project documents the full process of deploying a WordPress Web Solution on Amazon Web Services (AWS) using Red Hat Enterprise Linux. To keep the architecture both secure and scalable, I separated the web server from the database server, allowing WordPress to connect remotely to its backend database.

The process gave me hands-on practice with:
- Configuring and managing disks and volumes using LVM (Logical Volume Manager).
- Deploying a LAMP stack (Linux, Apache, MySQL/MariaDB, PHP) on Red Hat.
- Installing and configuring WordPress to integrate with a remote database server.
- Applying security best practices, such as restricting database access to only the web server’s private IP.

## **Step 1: Prepare the Web Server**

1. Launch a **Red Hat** EC2 instance that will serve as a **Web Server (WordPress)**.
- Configure Security Group to allow:
    - Port 22 → SSH access for management
    - Port 80 → HTTP web traffic


2. Create 3 EBS volumes (10GB each) in the same Availability Zone as your instance.
    
    **Direction**:
    -** At the EC2 menu bar, under Elastic Block Storage, click on → Volume → Create Volume.
    - Volume type: General Purpose SSD (gp3)
    - Size (GiB): 10
    - Availability zone: eu-north-1b

- Attach all 3 volumes one by one to your Web Server EC2 instance.

3. Log in to the Linux terminal using 'ec2-user' (since it is a Red Hat instance):
```bash
ssh -i <Your-private-key.pem> ec2-user@<EC2-Public-IP-address>

lsblk
# Inspect the block devices that are attached to the server.
```

**NOTE**: The image above shows that the 3 attached volumes are nvme0n1, nvme1n1 and nvme2n1.

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

**NOTE**: The new partitions can be easily identified by the p1 attached to each of them: nvme0n1p1, nvme1n1p1 and nvme2n1p1

4. Install LVM2 (Logical Volume Manager) to manage the 3 volumes as one pool and scan for disks/partitions available for LVM:
```bash
sudo yum install lvm2

sudo lvmdiskscan
```

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

5. Add all 3 PVs to a volume group (VG). Let’s name the group 'webdata-vg':
```bash
sudo vgcreate webdata-vg /dev/nvme0n1p1 /dev/nvme1n1p1 /dev/nvme2n1p1
```

- To confirm that the VG has been created, run:
```bash
sudo vgs
```

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

7. Verify the entire setup:
```bash
sudo vgdisplay -v
# Verifies that the VG, PVs, and LVs are correctly set up

sudo lsblk
```

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

- We need these mounts to persist after reboots, so let's update the /etc/fstab file:
```bash
sudo vi /etc/fstab
```
- Add UUIDs:
```bash
UUID=3d6418d2-2a5d-4809-8a93-834a4ad828d9   /var/www/html       ext4   defaults   0 0
UUID=9158c7e4-7fdb-4799-b1b3-c2a2ae74722f   /home/recovery/logs ext4   defaults   0 0
```

11. Test the configuration and reload the daemon:
```bash
sudo mount -a

sudo systemctl daemon-reload

df -h
# Verify your setup
```

## **Step 2: Prepare the Database Server**


1. Launch a second **Red Hat** EC2 instance that will serve as a DB Server.
- Configure Security Group to allow Port 22 (TCP): SSH access for management.


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

**NOTE**: The image above shows that the 3 attached volumes are nvme1n1, nvme2n1 and nvme3n1.

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

lsblk
# Verify the new partitions
```

3. Install LVM2 (Logical Volume Manager) to manage the 3 volumes as one pool and scan for disks/partitions available for LVM:
```bash
sudo yum install lvm2

sudo lvmdiskscan
```

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

4. Add all 3 PVs to a volume group (VG). Let’s name the group 'webdata-vg':
```bash
sudo vgcreate database-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1

sudo vgs
# Confirm that the VG has been created
```

5. Use lvcreate to create a single LV for the database:
```bash
sudo lvcreate -n db-lv -L 14G database-vg

sudo lvs
# Verify the logical volume has been created
```

6. Verify the entire setup:
```bash
sudo vgdisplay -v
# Verifies that the VG, PVs, and LVs are correctly set up

sudo lsblk
```

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

- Add UUIDs:
```bash
UUID=3419c338-dde5-4c1f-bd47-9e59d2c20f9d   /db   ext4   defaults   0 0
```

9. Test the configuration and reload the daemon:
```bash
sudo mount -a

sudo systemctl daemon-reload

df -h
#Verify your setup
```


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

- Start and enable PHP-FPM:
```bash
sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo systemctl status php-fpm
```

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


## **Step 4: Install MySQL (MariaDB) on the DB Server**
```bash
sudo yum update -y

sudo dnf install -y mariadb-server

sudo systemctl enable mariadb

sudo systemctl start mariadb

sudo systemctl status mariadb
```


## **Step 5: Configure Database for WordPress**

1. Set a root password and tighten security:
```bash
sudo mysql -u root -p
```

2. Create a database, a user, and grant permissions for the Web Server Private IP:
```bash
CREATE DATABASE wordpress_db;

CREATE USER `name_user`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'PasswordHere';

GRANT ALL ON wordpress_db.* TO 'name_user'@'<Web-Server-Private-IP-Address>';

FLUSH PRIVILEGES;

SHOW DATABASES;

exit
```

## **Step 6: Connect WordPress (web server) to the Remote Database**

1. Open MySQL/Aurora (port 3306) on the DB server EC2.
For extra security, use the Web Server’s private IP address (for secure DB access).


2. Install MYSQL client:
```bash
sudo dnf install -y mariadb

mysql -h <DB-Server-Private-IP> -u username -p

SHOW DATABASES;
```

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

## **Step 7: Access WordPress

- Access WordPress from your browser using: 'http://<Web-Server-Public-IP-Address>/'


*We have successfully set up WordPress on AWS using separate Web and DB servers. Your WordPress site is now ready for content creation and customization.*


# **🔧 Troubleshooting**

❌**Issue 1: EPEL 8 Package Error**
When trying to install PHP dependencies with EPEL 8 on my web server, I encountered an error when I used:
```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

✔️**Fix**:
- I verified my OS version using:
```bash
cat /etc/redhat-release
```
**The Result: Red Hat Enterprise Linux 10**

- I used the EPEL 9 package instead:
```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```


❌**Issue 2: MySQL Server Package Not Found**
Attempting to run 'sudo yum install mysql-server' failed on my Database server because RHEL 9 does not provide a package called 'mysql-server'.


✔️**Fix**:
- I installed MariaDB instead, which is fully compatible with WordPress.
```bash
sudo dnf install -y mariadb-server

sudo systemctl enable mariadb

sudo systemctl start mariadb
```
