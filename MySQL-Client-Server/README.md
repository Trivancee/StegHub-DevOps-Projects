# 🖥️ Implementing a Client-Server Architecture using MySQL

This project demonstrates how to implement a **Client-Server Architecture** using **MySQL** on AWS EC2 instances. Client-Server refers to an architecture where two or more computers communicate over a network.  
- The **Client** sends requests.  
- The **Server** responds to those requests. 

This architecture is widely used in systems where data is stored in a central location and multiple clients need to access or update it securely. Some common examples include:
- E-Commerce platforms
- Banking systems
- Healthcare management systems
- Ride-hailing apps (Uber, Bolt, Lyft)

In this setup:
- The **Server Instance** hosts the MySQL Database Server.
- The **Client Instance** has only the MySQL client installed, which connects remotely to the server.

---


##  Prerequisites

- Two Ubuntu EC2 Instances in the **same VPC** and **same Security Group**:
  - **Server Instance** → MySQL Server.
  - **Client Instance** → MySQL Client.
- Security Group Rules:
  - Allow **SSH (22)** → from your IP (for management).
  - Allow **MySQL (3306)** → from the **Client instance private IP** only.
- SSH key pair for authentication.  

---


## ⚙️ Step 1: Launch EC2 Instances

- Create **two Ubuntu EC2 instances** (Server & Client) in the same VPC and Security Group.  
- Configure Security Group to allow:
  - Port 22 (TCP) → SSH access from your IP
  - Port 3306 (TCP) → MySQL access from the Client’s private IP.  


---

## ⚙️ Step 2: Install MySQL on the Server

1. SSH into the **Server instance**:
```bash
ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

2. Update system packages:
```bash
sudo apt update && sudo apt upgrade -y
```

3. Install MySQL Server. This installation will make the server act as a Database server:
```bash
sudo apt install mysql-server -y
```

---

## ⚙️ Step 3: Configure MySQL Server for Remote Access

1. Open MySQL configuration file:
```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```
- Find the line 'bind-address = 127.0.0.1' and change it to:
```bash
bind-address = 0.0.0.0
```
This allows MySQL to accept connections from any IP address, not just localhost.

2. Restart MySQL to apply changes:
```bash
sudo systemctl restart mysql
# To apply the configuration changes
```

---

## ⚙️ Step 4: Create a Remote MySQL User

1. Log in to MySQL, create a user, and grant privileges (run each command separately):
```bash
sudo mysql

CREATE USER 'remote_user'@'<client-private-ip>' IDENTIFIED WITH mysql_native_password BY 'PassWord123!';

GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'<client-private-ip>' WITH GRANT OPTION;

FLUSH PRIVILEGES;

EXIT;
```

**Note**:
- Replace 'remote_user' with your desired remote username
- Replace '<client-private-ip>' with the client instance private IP
- Replace 'PassWord123!' with your unique password

---

## ⚙️ Step 5: Install MySQL Client on Client Instance

1. Connect to your client instance via SSH:
```bash
ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
```

2. Update and install MySQL client:
```bash
sudo apt update

sudo apt install mysql-client -y

mysql --version
# To verify the installation
```

---

## ⚙️ Step 6: Connect from MySQL Client to MySQL Server

1. Use the private IP of the server instance to connect (not the public IP) and enter the password you set:
```bash
mysql -u remote_user -p -h <server-private-ip>
```

2. Test the Connection:
```bash
SHOW DATABASES;
```


### 🎯 At this point:

✅ You have a **Client-Server Architecture** working with MySQL.
✅ The client can connect remotely to the server without SSH.
✅ You’ve restricted access securely (only the client server private IP address is allowed).

---


## 📊 Step 7: Perform Basic Database Operations

Now that the client is successfully connected to the MySQL server, the next step is to perform some fundamental database operations. These include creating a database, adding tables, inserting records, retrieving data, updating entries, deleting records, and finally dropping the database.

These operations demonstrate the full CRUD cycle (Create, Read, Update, Delete) and represent the foundation of how real-world applications interact with databases.

1. Create a Database (still on the client instance):
```bash
CREATE DATABASE Steg_Technology_Hub;

USE Steg_Technology_Hub;
```

2. Create a table (for this practical, let’s create an employee table):
```bash
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    role VARCHAR(50),
    salary DECIMAL(10,2)
);
```

- Insert data:
```bash
INSERT INTO employees (name, role, salary) 
VALUES 
('Ogechi Egbodo', 'Developer', 88000.00),
('Trivancy Media', 'Designer', 68000.00),
('Grace Nancy', 'Manager', 95000.00);
```

3. Query data:
```bash
SELECT * FROM employees;
# To get all rows

SELECT name, salary FROM employees WHERE role = 'Developer';
# To filter data
```

4. Update data:
```bash
UPDATE employees SET salary = 92000.00 WHERE name = 'Ogechi Egbodo';
```

5. Delete data:
```bash
DELETE FROM employees WHERE name = 'Grace Nancy';
```

6. Drop Table and Database:
```bash
DROP TABLE employees;

DROP DATABASE Steg_Technology_Hub
```

🎯 ### At this point:

✅ Created a **database**
✅ Created a **table**
✅ Inserted **records**
✅ Queried (retrieved) data
✅ Dropped and deleted structures
That’s a complete cycle of **basic DB operations**


---

# Conclusion

With this implementation, we successfully set up and tested a Client-Server Architecture using MySQL on AWS. This exercise not only shows how client-server communication works in practice but also builds a strong foundation for real-world applications where secure, centralized data management is critical.