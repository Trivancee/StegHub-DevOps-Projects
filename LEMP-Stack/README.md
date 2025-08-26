# 🌐 LEMP Stack Implementation with AWS 

This project demonstrates the deployment of a LEMP stack (Linux, Nginx, MySQL, PHP) on an AWS EC2 Ubuntu instance. The LEMP stack provides a reliable and scalable platform for hosting dynamic web applications, with Nginx serving as the high-performance web server and MySQL as the database management system. Similar to the LAMP stack, it is widely used in production environments to deliver fast, secure, and efficient web applications.

---

## 📌 Project Prerequisites

- **AWS EC2 instance**: Ubuntu 24.04 LTS (t3.micro Free Tier)
- **Security Group**: Inbound rules for  
  - Port 22 → SSH access  
  - Port 80 → HTTP web traffic  
- **SSH Key Pair**: For secure authentication  
- **GitHub**: Version control and documentation  
- **Visual Studio Code**: Code editor and SSH integration  
- **Web Browser**: For testing deployments
<img width="940" height="104" alt="Lemp Stack Server" src="https://github.com/user-attachments/assets/760d59fb-f650-4686-842c-b8c0ab8b9715" />

---

##  Step 1: Install Nginx Web Server
- SSH into the EC2 instance:
```bash
ssh -i /c/Users/USER/OneDrive/Documents/<Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

- Update and install Nginx:
```bash
sudo apt update
sudo apt install nginx
```

- Verify that nginx was successfully installed:
```bash
sudo systemctl status nginx
```
<img width="706" height="227" alt="Nginix running" src="https://github.com/user-attachments/assets/4ef36f95-61fe-4e44-98b9-e5519424ed40" />


- Test local access:
```bash
curl http://localhost:80
or
curl http://127.0.0.1:80
```

- Test public access using `http://<Public-IP>:80`

✅ Expected Result:
<img width="603" height="220" alt="Welcome" src="https://github.com/user-attachments/assets/cc0c12fd-82ef-4e87-8dbe-48fdcaf1e01d" />

---

##  Step 2: Install MySQL

```bash
sudo apt install mysql-server -y
sudo mysql
#To log into mysql
```

- Set root password and Secure MySQL installation::
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

exit

sudo mysql_secure_installation
#This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.
```

- Confirm Login into MySQL:
```bash
sudo mysql -p

exit
```
<img width="872" height="423" alt="Msql" src="https://github.com/user-attachments/assets/ba833fa9-f1c3-4e93-8b97-13a13bae3692" />

---

##  Step 3: Install PHP
```bash
sudo apt install php-fpm php-mysql
```
<img width="1474" height="507" alt="PHP" src="https://github.com/user-attachments/assets/ead188e3-cbdb-4eec-9efa-4578bd2792a9" />


##  Step 4: Configure Nginx for PHP
- Create web root and assign permissions:
```bash
sudo mkdir /var/www/projectLEMP

sudo chown -R $USER:$USER /var/www/projectLEMP
```

- Create config file:
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```

Paste this configuration:
```bash
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }
}
```

Save and Exit using: CTRL + O , **Enter** → then CTRL + X to **Exit**

Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

Test configuration:
```bash
sudo nginx -t
```
<img width="1159" height="256" alt="Syntax ok" src="https://github.com/user-attachments/assets/0124c206-b7e6-4b2c-a662-1fe787c8378b" />


- Disable default Nginix host and reload:
```bash
sudo unlink /etc/nginx/sites-enabled/default

sudo systemctl reload nginx
```

- Create index.html file (test page):
```bash
sudo echo 'Hello LEMP from hostname' > /var/www/projectLEMP/index.html
```
<img width="533" height="168" alt="Hello Lemp" src="https://github.com/user-attachments/assets/74600073-8993-4209-952e-2bf0eccd4e67" />

Access the site on your browser using `http://<Public-IP>:80`

✅ Expected result:
<img width="1497" height="328" alt="Expected Result" src="https://github.com/user-attachments/assets/211fdc21-f2e6-4cfa-aa16-2d984943cb3b" />


✅  At this point, the LEMP stack is completely installed and fully operational.

---

##  Step 5: Test PHP with Nginx

- Create a test PHP file:
```bash
nano /var/www/projectLEMP/info.php
```

Paste this code:
```bash
<?php
phpinfo();
```

Save and Exit.

Visit browser using `http://<Public-IP>/info.php`

✅ Expected result:
<img width="761" height="312" alt="PHP version" src="https://github.com/user-attachments/assets/855a0bef-893f-49b0-888c-9481e0b6e0f0" />


- Remove info.php after testing:
```bash
sudo rm /var/www/projectLEMP/info.php
```

---

##  Step 6: Create and retrieve data from MySQL database with PHP

To create a test database with a simple `To-do list` and configure access to it.

- Log into MySQL:
```bash
sudo mysql -p
```

- Create Database and User with Privileges:
```bash
CREATE DATABASE trivancee_database;

CREATE USER 'ogeeh_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

GRANT ALL ON trivancee_database.* TO 'ogeeh_user'@'%';

exit
```
<img width="485" height="59" alt="MYsql" src="https://github.com/user-attachments/assets/8f188925-35d6-405a-83e8-27f7fbdadada" />


- Confirm user permissions:
```bash
mysql -u ogeeh_user -p

SHOW DATABASES;
```
<img width="1167" height="721" alt="Database" src="https://github.com/user-attachments/assets/a0996028-1818-4152-8b6b-f1407bf52f19" />


- Create a test table named `todo_list`:
```bash
CREATE TABLE trivancee_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);
```

Insert sample rows: Note that this sample database is tailored to my experience as a Social media & Digital marketing strategist.

```bash
INSERT INTO trivancee_database.todo_list (content) 
VALUES ("Complete and submit marketing strategy for Odin Airbnb");

INSERT INTO trivancee_database.todo_list (content) 
VALUES ("Create a social media content calendar for Trivancy Media");

INSERT INTO trivancee_database.todo_list (content) 
VALUES ("Run a Facebook ads campaign targeting travelers in Accra");

INSERT INTO trivancee_database.todo_list (content) 
VALUES ("Track KFM Graduation Photoshoot campaign performance with Google Analytics and Meta Business Suite");
````

- Verify that the data was successfully saved:
```bash
SELECT * FROM trivancee_database.todo_list;

exit
```
<img width="1694" height="832" alt="Trivancee Database" src="https://github.com/user-attachments/assets/2a72a490-036e-431d-977a-4db7ae073cda" />

- Create a PHP script to retrieve data:
```bash
nano /var/www/projectLEMP/todo_list.php
```

Paste:
```bash
<?php
$user = "ogeeh_user";
$password = "PassWord.1";
$database = "trivancee_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```
Save and Exit.

- Access the page in your web browser using `http://<Public-IP>/todo_list.php`
- 
✅ Expected result:
<img width="584" height="200" alt="To do list" src="https://github.com/user-attachments/assets/d97b29ed-508b-4f97-b154-d0f9daa15a0b" />

With this guide, you’ve successfully set up a LEMP stack on AWS.


