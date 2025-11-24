# 🚀 DevSecOps CI/CD: Deploying a Secure Hotstar Clone

## 📌 **Overview**

This project demonstrates how to deploy a **Hotstar Clone** application on AWS using a **complete DevSecOps pipeline**. It integrates infrastructure automation, continuous integration, continuous deployment, and continuous security using tools like **Terraform, Jenkins, SonarQube, Docker, Kubernetes (EKS), Docker Scout, OWASP**, and AWS services.

---
## 🧱 Architecture Overview

<img width="785" height="436" alt="image" src="https://github.com/user-attachments/assets/31b2b770-f48a-4f05-8b62-9b4bf441e376" />

---

## 🧰 **Prerequisites**

* AWS Account
* Basic AWS Knowledge
* Understanding of DevSecOps Principles
* Familiarity With:

  * Docker
  * Jenkins
  * Java
  * SonarQube
  * AWS CLI
  * Kubectl
  * Terraform

---

# 🏗️ Step-by-Step Deployment Process

---

# **Step 1 — Setup AWS EC2 Instance & IAM Role**

## **1A: Launch EC2 Instance**

1. Open AWS Console → EC2
2. Launch Instance
3. Select **Ubuntu Server 24.04 LTS**
4. Choose **t2.large**
5. Add **30GB** storage
6. Configure Security Group:

   * Allow **SSH (22)**
   * Allow required app ports later
7. Launch with a key pair
8. Connect via SSH once ready

## **1B: Create IAM Role for EC2**

1. Go to **IAM**
2. Create Role → AWS Service → EC2
3. Attach **AdministratorAccess** (for learning only)
4. Name and create role
5. Attach role to EC2:

   * EC2 → Actions → Security → Modify IAM Role

---

# **Step 2 — Install Required Tools on EC2**

### Create script1.sh 

```bash
#!/bin/bash
sudo apt update -y
sudo apt install openjdk-17-jre -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

Run:

```bash
sudo chmod 777 script1.sh
./script1.sh
```

---

### Create script2.sh

```bash
#!/bin/bash
sudo apt update -y
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock

# Install kubectl
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

# Install Terraform
sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt-get install terraform -y

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```

Run:

```bash
sudo chmod 777 script2.sh
./script2.sh
```

---

### Install SonarQube

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Access UI:
👉 **http://EC2_PUBLIC_IP:9000**
Default login:

* **username:** admin
* **password:** admin

---

# **Step 3 — Jenkins CI/CD Configuration**

Install plugins:

* Terraform
* SonarQube Scanner
* NodeJS
* Eclipse Temurin (JDK)
* OWASP Dependency Check
* Docker
* Docker Pipeline
* Docker Scout

Add tools in **Manage Jenkins → Tools**:

* JDK 17
* Node 22
* Sonar Scanner
* OWASP 10.0.3
* Docker

---






































































































































































































































