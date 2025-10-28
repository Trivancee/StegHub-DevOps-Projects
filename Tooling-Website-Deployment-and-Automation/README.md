# 🚀 Tooling Website Deployment Automation with Continuous Integration (Jenkins)

In previous projects, I implemented horizontal scalability by introducing multiple web servers and a load balancer to efficiently distribute traffic. While managing a few servers manually is possible, scaling to dozens or hundreds quickly becomes repetitive and error-prone.

To address this challenge, this project introduces **automation** — a core DevOps principle that enhances speed, consistency, and agility in deployments. Using **Jenkins**, a popular open-source automation server, I automated the deployment of our Tooling Website, integrating continuous integration (CI) and continuous deployment (CD) practices.

<img width="531" height="312" alt="Jenkins Architecture" src="https://github.com/user-attachments/assets/916babea-9b67-43d0-97a6-a8797bb2cd77" />


## 🌐 Overview

**Continuous Integration (CI)** is a development strategy where developers frequently commit code in small increments. Each change triggers an automated build and test process, ensuring that new code integrates smoothly with the existing codebase.

In this project, Jenkins is configured to automatically pull code from a GitHub repository whenever changes occur and then deploy updated files to a central **NFS Server**. This forms the foundation of an automated CI/CD pipeline.


---

#  Step 1: Install Jenkins Server

1. Launch an Ubuntu EC2 instance and name it **Jenkins**
- Configure Security Group to allow:
    - Port 22 → SSH access for management
    - Port 8080 → Jenkins default HTTP port for the web user interface.

<img width="1369" height="387" alt="Jenkins Server" src="https://github.com/user-attachments/assets/bcdbbd99-7f72-4cdc-832c-8a4f207e7c1c" />

<img width="706" height="64" alt="Jenkins SG" src="https://github.com/user-attachments/assets/1b34030e-adb0-49ad-bcce-85c9b7958cf3" />


2. SSH into the server then update and install dependencies:
```bash
sudo apt update

sudo apt install -y fontconfig openjdk-17-jre
```

- Add the Jenkins repository key and source:
```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

- Install Jenkins:
```bash
sudo apt-get update

sudo apt-get install jenkins -y
```

- Start and enable Jenkins:
```bash
sudo systemctl start jenkins

sudo systemctl enable jenkins

sudo systemctl status jenkins
```

<img width="796" height="159" alt="Jenkins running" src="https://github.com/user-attachments/assets/609f023b-f8df-4e16-9e1d-448d1830a423" />


3. Access Jenkins:
    - Open your browser and go to: **`http://<Jenkins-server-public-ip>:8080`**
    - Then retrieve your initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

<img width="1682" height="1000" alt="Unlock Jenkins" src="https://github.com/user-attachments/assets/daba119c-7586-46e6-856a-cddb6a98b0e1" />

- Once you log in, select `Install Suggested Plugins` to install the Jenkins plugins
- Create an admin user (Username, Password, Full name, and Email):

<img width="753" height="424" alt="instance config" src="https://github.com/user-attachments/assets/b659a426-0b3a-441e-b66b-5d39a7f076d7" />

<img width="1334" height="534" alt="Jenkins ready PNG" src="https://github.com/user-attachments/assets/dd173507-163a-4d80-b13d-31cb4f237000" />


---


# Step 2 :  Configure Jenkins to retrieve source codes from GitHub using Webhooks

*Below are the steps to configure a simple Jenkins job/project that will be triggered by GitHub Webhooks and will execute a 'build' task to retrieve code from GitHub and store it locally on the Jenkins server.*

Enable Webhooks in your GitHub repository settings by following this guide:

1. Go to your "GitHub repository" > "Settings" > "Webhooks" > "Add Webhook" > "Payload URL" (**`http://<Jenkins-server-public-ip>:8080/github-webhook/`**) > "Content Type" (application/json) > Select “Just the push event” > Click on “Add Webhook”

<img width="1588" height="672" alt="Image 1" src="https://github.com/user-attachments/assets/f66fdbe9-9c18-4253-a4e2-10998299dac8" />

<img width="1191" height="826" alt="Image 2" src="https://github.com/user-attachments/assets/3c479cf1-3afb-4e64-9371-ded1c5e14a20" />


2. Go to Jenkins web console, click "**New Item**" and create a "**Freestyle project**"
- Item Name: tooling_github
- Select “OK”

<img width="1642" height="588" alt="New Item" src="https://github.com/user-attachments/assets/01e95442-5efb-4cd1-b18b-561587d89e5c" />

<img width="1650" height="870" alt="Image 3" src="https://github.com/user-attachments/assets/93523b8e-7493-426e-a4a4-a4a6321c7735" />


3. Go to your GitHub repository and copy your repository link. Mine is: [tooling](https://github.com/Trivancee/tooling). Note that my branch is **MAIN**, not master.

<img width="1588" height="631" alt="Image 6" src="https://github.com/user-attachments/assets/30a8f7a2-0158-426a-8d70-ae70aebb1238" />


4. In the configuration of the Jenkins freestyle project, choose "Source code management" > "Git" → > (Paste the Git repository link copied earlier) > Click on "Add" (to add credentials like username/password) > **Save** credentials
- Under “Branch Specifier,” select "Main" or "Master" depending on your GitHub repository branch
- "Save" the configuration.

<img width="1840" height="888" alt="Image 5" src="https://github.com/user-attachments/assets/28543c8b-74dc-445d-a2e7-08ed85332027" />

<img width="907" height="432" alt="Image 4" src="https://github.com/user-attachments/assets/466ea41f-f429-4340-900d-85d75c5805fd" />


-  Click the "Build Now" button. If you have configured everything correctly, the build will be successful.

<img width="1230" height="886" alt="step 7 Before" src="https://github.com/user-attachments/assets/849427d5-4b92-45a9-b95e-0d088b3777a5" />

<img width="1377" height="715" alt="Image 8 After" src="https://github.com/user-attachments/assets/d2a944c1-0a4f-40a1-bb44-f839f81267a2" />


5. Configure for automated builds:
- Go to "Configure" > Triggers > Select “GitHub hook trigger for GitScm polling"
- Go to Post-build actions > Select “Archive the artifacts”.
- Type ** under files to archive and then **Save**.

<img width="1582" height="561" alt="Image 10" src="https://github.com/user-attachments/assets/37e83034-35a8-4b70-9183-c03c5093f7d0" />

<img width="1644" height="738" alt="Image 9" src="https://github.com/user-attachments/assets/60e69603-6d43-4327-83ff-8212beaef2eb" />


6. Now, make a change to the Readme.md file of your GitHub repository (e.g., edit the README.MD file) and push it to the main branch. You will see that a new build has been launched automatically (by webhook), and you can see its results - artifacts, saved on the Jenkins server.

<img width="1374" height="842" alt="Push Successful" src="https://github.com/user-attachments/assets/deae9b35-91c3-4b38-8644-2c64b3d2b872" />


*We have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and file transfer is initiated by GitHub).*


- By default, artifacts are stored locally on the Jenkins server:
```bash
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```

<img width="1194" height="98" alt="image (5)" src="https://github.com/user-attachments/assets/0fcd0486-0c90-4b91-8ab0-91b795199a7a" />


# Step 3: Configure Jenkins to copy files to NFS server via SSH

Now that we have our artifacts saved locally on the Jenkins server, let's set up Jenkins to copy them to our NFS server. We will use the "Publish Over SSH" plugin for this task.

1. Install the Publish Over SSH Plugin:
- On the Jenkins dashboard, go to "Settings (Manage Jenkins)" > "Plugins" > “Available Plugins”
- Search for "Publish over SSH" and install it without restarting.

<img width="1832" height="480" alt="Manage Jenkins" src="https://github.com/user-attachments/assets/ce18f3c8-f5fa-4c84-a210-42225ec6c548" />


2. Configure Jenkins to copy artifacts to the NFS server:
- On the main dashboard, go to "Manage Jenkins (settings)" > "System"
- Scroll down to the "Publish over SSH" section and configure the Plugin to connect to your NFS server.
    - **Key**: Provide your private key used to connect to the NFS server via SSH
    - **SSH Server Name:** NFS-Server
    - **Hostname**: The private IP address of your NFS server
    - **Username**: ec2-user
    - **Remote directory**: /mnt/apps
- Test the configuration and make sure the connection returns **Success.**

<img width="1132" height="876" alt="Image 11" src="https://github.com/user-attachments/assets/d81c05da-f526-45cf-ae37-53bf18957bb2" />

<img width="1814" height="894" alt="Image 12" src="https://github.com/user-attachments/assets/06b71c3e-f924-4de2-80a1-e7d0583091b7" />


3. Open the Jenkins job/project configuration page and add another "Post-build Action”.
- On the main dashboard, go to "Manage Jenkins (settings)" > "System"
- Scroll down to the "Publish over SSH" section and click on “Add Post-build Action”
- Select “Send build artifacts over SSH”
    - **Source file**: Type **
    - **Key**: Replace the previous key with the **new private key** generated on the Jenkins server (Check my troubleshooting section on how to get it).
- Then **"Save"**

<img width="1351" height="811" alt="Image 13" src="https://github.com/user-attachments/assets/f0633e64-0221-48cb-95ce-f8fad6072fff" />

<img width="1182" height="892" alt="Image 14" src="https://github.com/user-attachments/assets/f813fc1e-6541-48a5-b776-33235c384068" />


- Go ahead, change something in README.MD file in your GitHub Tooling repository.

<img width="1684" height="862" alt="Push Worked" src="https://github.com/user-attachments/assets/9b12427f-9c33-4330-8808-02260f0503b9" />

<img width="758" height="188" alt="GitHub Webhook" src="https://github.com/user-attachments/assets/6231d571-4254-46c4-899a-8dfc6d9bac6a" />

If you see the changes you previously made in your GitHub repository, congratulations! Your Jenkins job is working as expected.


4. To make sure that the files in /mnt/apps have been updated, run this in your NFS Server. The readme file should appear: 
```bash
cat /mnt/apps/README.md
```

<img width="1680" height="715" alt="Terminal result" src="https://github.com/user-attachments/assets/82dd2cf6-68bd-4527-908d-65288f93c270" />

This setup now automatically deploys your code changes from GitHub to your NFS server, thus streamlining your development workflow and enabling continuous integration.



---


# 🧾 Troubleshooting: Permission Denied During Jenkins SSH Deployment

### 🧩 Problem

After setting up the **Publish Over SSH** plugin in Jenkins and configuring it to connect to the **NFS Server**, the build trigger from GitHub succeeded, but Jenkins logs showed this error:

<img width="1632" height="864" alt="Permission denied" src="https://github.com/user-attachments/assets/274e2cfb-d24c-4055-89e3-6fb565d2fb37" />


### 🧩 Root Cause

There were **two main issues**:

1. **Authentication mismatch:**
    
    Jenkins was using an **AWS private key** (used for EC2 login) in the *Publish over SSH* configuration, which didn’t match the public key stored on the NFS server for the Jenkins user. This caused inconsistent authentication between Jenkins and the NFS target.
    
2. **Permission issue on the NFS directory:**
    
    Even after successful SSH authentication, Jenkins could not write to `/mnt/apps` because the directory was **owned by root**, and the SSH user (`ec2-user`) had **no write permissions**.
    

### 🧰 **Steps Taken to Fix the Issue**

**1. Generated a New SSH Key on the Jenkins Server**

Logged into the Jenkins server:
```bash
sudo su - jenkins
ssh-keygen -t rsa -b 4096 -C "jenkins@steg-hub"
cat ~/.ssh/id_rsa
```

**2. Added the New Public Key to the NFS Server**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
sudo vi ~/.ssh/authorized_keys
```

Then pasted the Jenkins public key (`id_rsa.pub`) into the file and saved it.

- Ensured proper permissions:
```bash
chmod 600 ~/.ssh/authorized_keys
```

**3. Updated Jenkins SSH Configuration**

- In Jenkins UI → **Manage Jenkins (settings) → System → Publish over SSH**
- Removed the old AWS private key.
- Replaced it with the **new private key** generated on the Jenkins server (`id_rsa`).
- Saved the configuration and tested the connection — it returned **Success** ✅

**4. Verified SSH Access from Jenkins to NFS**

On the Jenkins server:
```bash
ssh -i ~/.ssh/id_rsa ec2-user@<NFS-Server-Private-IP>
```

Login worked without a password prompt.

Then tested writing to the deployment directory:
```bash
ssh -i ~/.ssh/id_rsa ec2-user@10.0.0.117 'touch /mnt/apps/testfile'
```

**Result**:

```bash
touch: cannot touch '/mnt/apps/testfile': Permission denied
```

**→ Authentication issue was fixed, but permissions were still restricted.**


**5. Fixed Directory Permissions on NFS Server**
```bash
sudo chown -R ec2-user:ec2-user /mnt/apps
sudo chmod -R 755 /mnt/apps
```


**6. Re-Tested the Deployment**

From Jenkins:
```bash
ssh -i ~/.ssh/id_rsa ec2-user@<NFS-Server-Private-IP> 'touch /mnt/apps/testfile'
```


✅ File created successfully.

Re-ran the Jenkins job, and it completed with:

<img width="1684" height="862" alt="Worked" src="https://github.com/user-attachments/assets/5efbd502-262a-4355-a584-18ca8c2e8241" />



### ✅ **Outcome**

- Jenkins now **authenticates properly** using its own key pair.
- Jenkins can **transfer build artifacts** to the NFS server successfully.
- Directory `/mnt/apps` has correct ownership and permissions for deployment.
- Builds triggered via GitHub webhook now complete with a **SUCCESS** status.
