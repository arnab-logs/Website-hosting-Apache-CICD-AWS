---

# CI/CD Pipeline to Deploy a Static Website on Apache using Jenkins and AWS EC2

## Introduction

This project demonstrates how to create a **CI/CD pipeline** using **Jenkins** to automatically deploy a static website on an **Apache web server** hosted on an **AWS EC2 instance**.
It’s designed to help beginners understand not just the *steps* but also the *reasoning* behind each one, so you can connect the dots and see how everything fits together.

By the end of this guide, we’ll have a running website that updates automatically every time we push code to GitHub.

---

## Project Overview

We’ll be working on an **Ubuntu-based EC2 instance** named:

```
Arnab-CICD-Website-Apache-Jenkins
```

<img width="1926" height="1548" alt="image" src="https://github.com/user-attachments/assets/890f1305-70cb-4446-8434-a956819b8376" />

Our goal is to:

1. Install and configure **Apache** to host a simple static website.
2. Install and configure **Jenkins** to automate deployment.
3. Connect Jenkins with a **GitHub repository** containing the website’s code.
4. Use **GitHub webhooks** to trigger automatic redeployments on every code update.

---

## Prerequisites

Before we begin, make sure you have:

* An AWS EC2 instance (Ubuntu) up and running.
* Security group configured to allow:

  * **Port 22** (SSH)
  * **Port 80** (HTTP for Apache)
  * **Port 8080** (for Jenkins)
 
* A GitHub repository containing your static website code.

---

## Step-by-Step Setup on Ubuntu EC2

### Step 1: Update System Packages

Before installing any software, always update your package list to get the latest versions.

```bash
sudo apt update -y
sudo apt upgrade -y
```

This ensures your system is secure and ready for installations.

---

### Step 2: Install Apache Web Server

Apache is the web server that will host your static website.

```bash
sudo apt install apache2 -y
```

Once installed, enable Apache to start automatically on boot and start it immediately:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

To confirm Apache is running:

```bash
sudo systemctl status apache2
```

<img width="1928" height="658" alt="image" src="https://github.com/user-attachments/assets/c221c50d-c23a-4a11-92bf-8695b512f55b" />

Press `q` to exit the status screen.

---

### Step 3: Verify Apache Installation

Open your browser and visit:

```
http://<your-ec2-public-ip>
```

You should see the **“Apache2 Ubuntu Default Page.”**

<img width="2938" height="1746" alt="image" src="https://github.com/user-attachments/assets/4f58ebd0-e68f-4821-bc83-82e1b6216682" />

If you see that, it means Apache is running successfully.

---

## Jenkins Installation and Setup

Jenkins will be the automation engine of our CI/CD pipeline. It monitors our GitHub repository and deploys new changes automatically.

---

### Step 4: Update Packages Before Installing Jenkins

Always update your packages before adding new software:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 5: Install Java (Required for Jenkins)

Jenkins runs on Java, so install OpenJDK 17 (the recommended version):

```bash
sudo apt install fontconfig openjdk-17-jre -y
```

Verify the installation:

```bash
java -version
```

Expected output:

```
openjdk version "17.0.x" ...
```

---

### Step 6: Add Jenkins Repository and Key

To ensure you get the latest stable version of Jenkins, add its official repository and key:

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Then, update your package list:

```bash
sudo apt update
```

---

### Step 7: Install Jenkins

Now install Jenkins:

```bash
sudo apt install jenkins -y
```

Start and enable Jenkins to run automatically on boot:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Verify Jenkins status:

```bash
sudo systemctl status jenkins
```

<img width="2940" height="834" alt="image" src="https://github.com/user-attachments/assets/9f32d77b-6dee-4a58-b87e-485ac27192bc" />

You should see the service as **active (running)**.

---

### Step 8: Configure AWS Security Group for Jenkins

In your EC2 Management Console:

1. Go to **Security Groups → Inbound Rules**.
2. Add the following rules:

| Type            | Protocol | Port Range | Source    |
| --------------- | -------- | ---------- | --------- |
| HTTP            | TCP      | 80         | 0.0.0.0/0 |
| Custom TCP Rule | TCP      | 8080       | 0.0.0.0/0 |

This will allow access to both Apache (port 80) and Jenkins (port 8080).

<img width="2940" height="1186" alt="image" src="https://github.com/user-attachments/assets/62de920b-25c7-4b5b-bc81-9227e1055aea" />

---

### Step 9: Access Jenkins

In your browser, go to:

```
http://<your-ec2-public-ip>:8080
```
<img width="2940" height="1694" alt="image" src="https://github.com/user-attachments/assets/b5994184-a606-4b41-984b-8d756dfc33db" />

You’ll see the Jenkins unlock page asking for an administrator password.
Retrieve it using:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy and paste it into the browser.
Follow the on-screen steps to:

* Install suggested plugins
* Create an admin user
* Finish the initial setup

<img width="2938" height="1746" alt="image" src="https://github.com/user-attachments/assets/2c184c59-0e2f-46c9-b1db-1091e989a265" />

You should now be able to access your Jenkins dashboard.

---

## Configuring the Jenkins Project

Once Jenkins is set up, let’s create a job to automate our website deployment.

1. Click **“New Item”** on the Jenkins dashboard.
2. Name it **Jenkins-CICD-AWS**.
3. Select **Freestyle Project**.
4. Add a description:

<img width="2938" height="918" alt="image" src="https://github.com/user-attachments/assets/e5c7cee7-7f68-48f8-b9ea-3b5b428b94f5" />

   ```
   CICD Project for static website
   ```

---

### Step 10: Connect Jenkins to Your GitHub Repository

In the **Source Code Management** section:

* Select **Git**.
* Enter the repository URL:

  ```
  https://github.com/arnab-logs/Website-hosting-Apache-CICD-AWS.git
  ```
* Since this is a public repository, credentials are not required.
* Set the branch to:

  ```
  main
  ```
<img width="2938" height="1338" alt="image" src="https://github.com/user-attachments/assets/78abdde4-9723-4cb5-b9cd-ae16382ffbcd" />

Under **Build Triggers**, select:

* **GitHub hook trigger for GITScm polling**

Click **Save**.

Now, click **Build Now** to test the job.
If successful, you’ll see a “Build #1” with a green checkmark.

<img width="2938" height="1472" alt="image" src="https://github.com/user-attachments/assets/5ed0ad83-bac7-4ce0-b617-a0d86f7a487a" />

Inside Jenkins, the workspace for this job will be created at:

```
/var/lib/jenkins/workspace/Jenkins-CICD-AWS
```

---

### Step 11: Verify Workspace Files

Go to your EC2 terminal and navigate to the workspace:

```bash
cd /var/lib/jenkins/workspace/Jenkins-CICD-AWS
```

<img width="1498" height="460" alt="image" src="https://github.com/user-attachments/assets/9dd7f4a5-1ff5-4fdc-8eda-6d1e3cc44643" />

You should see the source code files pulled from your GitHub repository.

This confirms that Jenkins successfully fetched the code.

---

## Linking Jenkins with Apache

Now that Jenkins has the latest website code, we’ll configure it to deploy that code to Apache’s default directory.

The Apache web root directory is:

```
/var/www/html
```

---

### Step 12: Add Build Step in Jenkins

Go back to your Jenkins project → **Configure** → **Build Steps** → **Execute Shell**.
Add the following commands:

```bash
rm -rf /var/www/html/*
cp -r * /var/www/html/
```

Here’s what they do:

* The first command removes old website files from `/var/www/html/`.
* The second command copies the new website files from Jenkins workspace to Apache’s directory.

Save the configuration and click **Build Now**.

---

### Step 13: Fix Permission Errors (If Any)

The first build may fail due to permission issues.
To fix this, run the following commands on your EC2 terminal:

```bash
sudo usermod -aG www-data jenkins
sudo chown jenkins:www-data /var/www/html
```

These commands:

* Add the `jenkins` user to the `www-data` group (Apache’s group).
* Change ownership of the `/var/www/html` directory so Jenkins can access it.

After applying these permissions, click **Build Now** again.
This time, the build should succeed.

Visit your EC2 public IP:

```
http://<your-ec2-public-ip>
```

<img width="2940" height="1542" alt="image" src="https://github.com/user-attachments/assets/62e2cf7d-4599-4cdd-9f71-b72ec08ea6a8" />

You should now see your website successfully deployed!

---

## Automating Deployments with GitHub Webhooks

Now let’s automate the process — so any change pushed to GitHub automatically triggers Jenkins to build and redeploy.

---

### Step 14: Add a GitHub Webhook

1. Go to your GitHub repository → **Settings → Webhooks → Add webhook**

2. Enter your Jenkins endpoint in the **Payload URL** field:

   ```
   http://<your-ec2-public-ip>:8080/github-webhook/
   ```

3. Set the **Content type** to `application/json`.

<img width="1536" height="594" alt="image" src="https://github.com/user-attachments/assets/83d1c5f6-b928-4ea2-9371-df01505a12be" />

4. Click **Add Webhook**.

This webhook will notify Jenkins whenever new code is pushed.

---

### Step 15: Test the Webhook Integration

Make a small change in your GitHub repository — for example, edit the `contact.html` file and update some text or phone number.

<img width="2940" height="1046" alt="image" src="https://github.com/user-attachments/assets/821a8226-8805-45d1-89ec-7163f92b7cad" />

<img width="2940" height="1046" alt="image" src="https://github.com/user-attachments/assets/0ec5e546-ee76-4e18-8c1c-a29b4b1cc823" />

Commit and push your changes.

Jenkins will automatically trigger a new build, pull the updated code, and redeploy it to Apache.
When you refresh your website, the changes should appear instantly.

<img width="2940" height="1514" alt="image" src="https://github.com/user-attachments/assets/488d7cb0-0df6-4821-ab5b-872393460a27" />

---

## Conclusion

We’ve now built a **complete CI/CD pipeline** that automatically deploys a static website using **Jenkins**, **Apache**, and **AWS EC2**.

Here’s what we achieved:

* Installed and configured Apache and Jenkins on Ubuntu EC2.
* Connected Jenkins to a GitHub repository.
* Automated deployments using build steps and permissions.
* Integrated webhooks to enable continuous delivery.

This setup introduces the core principles of DevOps — automation, integration, and continuous improvement — in a simple, practical way that beginners can easily follow and expand upon.

---

**Author:** Arnab Nandi
**Project:** CI/CD Pipeline for Static Website Deployment using Jenkins, Apache, and AWS EC2

---
