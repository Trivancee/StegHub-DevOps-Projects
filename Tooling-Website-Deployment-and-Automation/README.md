# 🚀 Tooling Website Deployment Automation with Continuous Integration (Jenkins)

In previous projects, I implemented horizontal scalability by introducing multiple web servers and a load balancer to efficiently distribute traffic. While managing a few servers manually is possible, scaling to dozens or hundreds quickly becomes repetitive and error-prone.

To address this challenge, this project introduces **automation** — a core DevOps principle that enhances speed, consistency, and agility in deployments. Using **Jenkins**, a popular open-source automation server, I automated the deployment of our Tooling Website, integrating continuous integration (CI) and continuous deployment (CD) practices.


## 🌐 Overview

**Continuous Integration (CI)** is a development strategy where developers frequently commit code in small increments. Each change triggers an automated build and test process, ensuring that new code integrates smoothly with the existing codebase.

In this project, Jenkins is configured to automatically pull code from a GitHub repository whenever changes occur and then deploy updated files to a central **NFS Server**. This forms the foundation of an automated CI/CD pipeline.


---

#  Step 1: Install Jenkins Server

1. Launch an Ubuntu EC2 instance and name it **Jenkins**
- Configure Security Group to allow:
    - Port 22 → SSH access for management
    - Port 8080 → Jenkins default HTTP port for the web user interface.


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

3. Access Jenkins:
    - Open your browser and go to: **`http://<Jenkins-server-public-ip>:8080`**
    - Then retrieve your initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Once you login, select `Install Suggested Plugins` to install the Jenkins plugins
- Create an admin user (Username, Password, Full name and Email):

---


# Step 2 :  Configure Jenkins to retrieve source codes from GitHub using Webhooks

*Below are the steps to configure a simple Jenkins job/project that will be triggered by GitHub Webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.*

Enable Webhooks in your GitHub repository settings by following this guide:

1. Go to your "GitHub repository" > "Settings" > "Webhooks" > "Add Webhook" > "Payload URL" (**`http://<Jenkins-server-public-ip>:8080/github-webhook/`**) > "Content Type" (application/json) > Select “Just the push event” > Click on “Add Webhook”



2. Go to Jenkins web console, click "**New Item**" and create a "**Freestyle project**"
- Item Name: tooling_github
- Select “OK”


3. Go to you GitHub repository and copy your repository link. Mine is: [tooling](https://github.com/Trivancee/tooling) . Note that my branch is **MAIN** not master.



4. In configuration of the Jenkins freestyle project choose "Source code management" > "Git" → > (Paste the Git repository link copied earlier) > Click on "Add" (to add credential like username/password) > **Save** credential
- Under “Branch Specifier” select "Main" or "Master" depending on your GitHub repository branch
- "Save" the configuration.



-  Click "Build Now" button, if you have configured everything correctly, the build will be successful.



5. Configure for automated builds:
- Go to "Configure" > Triggers > Select “GitHub hook trigger for GITScm polling"
- Go to Post-build actions > Select “Archive the artifacts”.
- Type ** under files to archive and then **Save**.


6. Now, make a change to the Readme.md file of your GitHub repository (e.g., edit the README.MD file) and push it to the main branch. You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.



*We have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub).*


- By default, artifacts are stored locally on the Jenkins server:
```bash
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```


# Step 3: Configure Jenkins to copy files to NFS server via SSH

Now that we have our artifacts saved locally on the Jenkins server, let's set up Jenkins to copy them to our NFS server. We will use the "Publish Over SSH" plugin for this task.

1. Install the Publish Over SSH Plugin:
- On the Jenkins dashboard, go to "Settings (Manage Jenkins)" > "Plugins" > “Available Plugins”
- Search for "Publish over SSH" and install it without restarting.




2. Configure Jenkins to copy artifacts to NFS server:
- On the main dashboard, go to "Manage Jenkins (settings)" > "System"
- Scroll down to the "Publish over SSH" section and configure the Plugin to connect to your NFS server.
    - **Key**: Provide you private key used to connect to NFS server via SSH
    - **SSH Server Name:** NFS-Server
    - **Hostname**: The private IP address of your NFS server
    - **Username**: ec2-user
    - **Remote directory**: /mnt/apps
- Test the configuration and make sure the connection returns **Success.**



3. Open the Jenkins job/project configuration page and add another "Post-build Action”.
- On the main dashboard, go to "Manage Jenkins (settings)" > "System"
- Scroll down to the "Publish over SSH" section and click on “Add Post-build Action”
- Select “Send build artifacts over SSH”
    - **Source file**: Type **
    - **Key**: Replace the previous key with the **new private key** generated on the Jenkins server (Check my troubleshooting section on how to get it).
- Then **"Save"**



- Go ahead, change something in README.MD file in your GitHub Tooling repository.


If you see the changes you previously made in your GitHub repository, congratulations! Your Jenkins job is working as expected.



4. To make sure that the files in /mnt/apps have been updated, run this in your NFS Server. The readme file should appear: 
```bash
cat /mnt/apps/README.md
```


This setup now automatically deploys your code changes from GitHub to your NFS server, thus, streamlining your development workflow and enabling continuous integration.



---


# 🧾 Troubleshooting: Permission Denied During Jenkins SSH Deployment

### 🧩 Problem

After setting up the **Publish Over SSH** plugin in Jenkins and configuring it to connect to the **NFS Server**, the build trigger from GitHub succeeded, but Jenkins logs showed this error:




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

Re-ran the Jenkins job and it completed with:


### ✅ **Outcome**

- Jenkins now **authenticates properly** using its own key pair.
- Jenkins can **transfer build artifacts** to the NFS server successfully.
- Directory `/mnt/apps` has correct ownership and permissions for deployment.
- Builds triggered via GitHub webhook now complete with a **SUCCESS** status.