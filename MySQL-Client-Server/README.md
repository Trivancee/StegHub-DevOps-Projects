# 🖥️ Implementing a Client-Server Architecture using MySQL

This project demonstrates how to implement a **Client-Server Architecture** using **MySQL** on AWS EC2 instances. Client-server refers to an architecture where two or more computers communicate over a network.  
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
<img width="940" height="102" alt="MySQL Server and Client Instances" src="https://github.com/user-attachments/assets/62a0d2bc-8318-4c6a-ad2c-18306a0b9b78" />

---


## ⚙️ Step 1: Launch EC2 Instances

- Create **two Ubuntu EC2 instances** (Server & Client) in the same VPC and Security Group.  
- Configure Security Group to allow:
  - Port 22 (TCP) → SSH access from your IP
  - Port 3306 (TCP) → MySQL access from the Client’s private IP.  
<img width="1536" height="400" alt="Inbound Rules" src="https://github.com/user-attachments/assets/96cf1a37-f295-496c-872d-96010ccea1f2" />


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
<img width="1098" height="595" alt="Bind Address" src="https://github.com/user-attachments/assets/40106ef8-e93a-4318-b82d-cc3122af9dbc" />

2. Restart MySQL to apply changes:
```bash
sudo systemctl restart mysql
# To apply the configuration changes
```


## ⚙️ Step 4: Create a Remote MySQL User

1. Log in to MySQL, create a user, and grant privileges (run each command separately):
```bash
sudo mysql

CREATE USER 'remote_user'@'<client-private-ip>' IDENTIFIED WITH mysql_native_password BY 'PassWord123!';

GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'<client-private-ip>' WITH GRANT OPTION;

FLUSH PRIVILEGES;

EXIT;
```
<img width="1036" height="327" alt="Creating remote user" src="https://github.com/user-attachments/assets/b2a49027-fdf1-4f9e-a252-6361d001bb54" />

**Note**:
- Replace 'remote_user' with your desired remote username
- Replace '<client-private-ip>' with the client instance private IP
- Replace 'PassWord123!' with your unique password


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


## ⚙️ Step 6: Connect from MySQL Client to MySQL Server

1. Use the private IP of the server instance to connect (not the public IP) and enter the password you set:
```bash
mysql -u remote_user -p -h <server-private-ip>
```
<img width="1080" height="349" alt="MySQL login successful" src="https://github.com/user-attachments/assets/6b1da34b-8024-464c-8fe7-577b63dac2a6" />

2. Test the Connection:
```bash
SHOW DATABASES;
```
<img width="736" height="300" alt="Database" src="https://github.com/user-attachments/assets/08cfedc4-517e-4722-b3eb-ec3376c55924" />


### 🎯 At this point:

- You have a **Client-Server Architecture** working with MySQL.
- The client can connect remotely to the server without SSH.
- You’ve restricted access securely (only the client server private IP address is allowed).

---


## 📊 Step 7: Perform Basic Database Operations

Now that the client is successfully connected to the MySQL server, the next step is to perform some fundamental database operations. These include creating a database, adding tables, inserting records, retrieving data, updating entries, deleting records, and finally dropping the database.

These operations demonstrate the full CRUD cycle (Create, Read, Update, Delete) and represent the foundation of how real-world applications interact with databases.

1. Create a Database (still on the client instance):
```bash
CREATE DATABASE Steg_Technology_Hub;

USE Steg_Technology_Hub;
```
<img width="840" height="301" alt="Steghub database" src="https://github.com/user-attachments/assets/7a7b21a1-7e53-455c-b67b-29eea9edf4c5" />


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
<img width="892" height="474" alt="Basic database operation" src="https://github.com/user-attachments/assets/31e516ba-193a-4007-8c3b-cc68dd3417ba" />


3. Query data:
```bash
SELECT * FROM employees;
# To get all rows

SELECT name, salary FROM employees WHERE role = 'Developer';
# To filter data
```
<img width="960" height="462" alt="Quering Database" src="https://github.com/user-attachments/assets/c8a6fe07-1b47-4316-91d6-79f6b5a49cf4" />


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

### 🎯 At this point:

- Created a **database**
- Created a **table**
- Inserted **records**
- Queried (retrieved) data
- Dropped and deleted structures

That’s a complete cycle of **basic DB operations**

# Conclusion

With this implementation, we successfully set up and tested a Client-Server Architecture using MySQL on AWS. This exercise not only shows how client-server communication works in practice but also builds a strong foundation for real-world applications where secure, centralized data management is critical.
