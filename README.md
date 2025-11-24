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
  * Docker Scout

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
* Docker Commons
* Docker API
* Docker-build-step

Add tools in **Manage Jenkins → Tools**:

* JDK 17
* Node 22
* Sonar Scanner
* OWASP 10.0.3
* Docker

---

## **3A — EKS Provisioning Job (Terraform)**

That is done now go to Jenkins and add a terraform plugin to provision the AWS EKS using the Pipeline Job.

Go to Jenkins dashboard –> Manage Jenkins –> Plugins

Available Plugins, Search for Terraform and install it.

<img width="748" height="160" alt="image" src="https://github.com/user-attachments/assets/5f1df44f-ff65-40a3-9a54-5fd520408ca0" />

---
let’s find the path to our Terraform (we will use it in the tools section of Terraform)

## Note:-> On your EC2 install type `which terraform`

Now come back to Manage Jenkins –> Tools

Add the terraform in Tools

<img width="744" height="520" alt="image" src="https://github.com/user-attachments/assets/14ae0fb9-df14-4275-9a5b-07c053915b05" />

---
Apply and save.

CHANGE YOUR S3 BUCKET NAME IN THE BACKEND.TF

I want to do this with build parameters to apply and destroy while building only.

you have to add this inside job like the below image

<img width="788" height="455" alt="image" src="https://github.com/user-attachments/assets/2f8cdb56-05c1-4bde-b917-68602f7c2ebd" />

---

Add pipeline:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/linuxdocs2025/Project-Hotstar-Clone.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('EKS_TERRAFORM') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('EKS_TERRAFORM') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('EKS_TERRAFORM') {
                    sh 'terraform plan'
                }
            }
        }

        stage('Terraform Apply/Destroy') {
            steps {
                dir('EKS_TERRAFORM') {
                    sh "terraform ${action} --auto-approve"
                }
            }
        }
    }
}
```


let’s apply and save and Build with parameters and select action as apply

<img width="685" height="207" alt="image" src="https://github.com/user-attachments/assets/a253ba28-9aaa-497e-ba07-f52ba21688e9" />

Stage view it will take max 10mins to provision

<img width="760" height="314" alt="image" src="https://github.com/user-attachments/assets/62dca8bc-c0fd-4808-b650-5799dd491052" />

Check in Your Aws console whether it created EKS or not.

<img width="740" height="184" alt="image" src="https://github.com/user-attachments/assets/0cc4b925-0a3d-497c-9522-01687f1a46b1" />

Ec2 instance is created for the Node group

<img width="742" height="129" alt="image" src="https://github.com/user-attachments/assets/1a11ca2a-dd9a-453b-b0f4-a62a9aea49d3" />

---

## **3B — Hotstar Build Pipeline**

### Configure SonarQube Token

Add in Jenkins Credentials → **Secret Text**

### Configure Docker Credentials

Add DockerHub credentials as "docker"

### Configure in Global Tool Configuration

Go to Manage Jenkins → Tools → Install JDK(17) and NodeJs (22) → Click on Apply and Save

In the Sonarqube Dashboard add a quality gate also

Administration–> Configuration–>Webhooks

<img width="736" height="259" alt="image" src="https://github.com/user-attachments/assets/941b4c69-1b3c-4860-9c8f-03d8d307a93e" />

Click on Create

Add details

```
#in url section of quality gate
http://jenkins-public-ip:8080/sonarqube-webhook/
```

### Install Docker Scout

```bash
docker login
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
```

---

### Full CI/CD Pipeline

```groovy
pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node22'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/linuxdocs2025/Project-Hotstar-Clone.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Hotstar \
                        -Dsonar.projectKey=Hotstar
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps { sh "npm install" }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                                odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('DockerScout File Scan') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker-scout quickview fs://.'
                        sh 'docker-scout cves fs://.'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t hotstar ."
                        sh "docker tag hotstar <your-dockerhub-username>/hotstar:latest"
                        sh "docker push <your-dockerhub-username>/hotstar:latest"
                    }
                }
            }
        }

        stage('DockerScout Image Scan') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker-scout quickview <your-dockerhub-username>/hotstar:latest'
                        sh 'docker-scout cves <your-dockerhub-username>/hotstar:latest'
                        sh 'docker-scout recommendations <your-dockerhub-username>/hotstar:latest'
                    }
                }
            }
        }

        stage('Deploy Docker Locally') {
            steps {
                sh "docker run -d --name hotstar -p 3000:3000 <your-dockerhub-username>/hotstar:latest"
            }
        }
    }
}
```

---

Deploy to Container

< EC2-public-ip:3000 >

Output

<img width="904" height="371" alt="image" src="https://github.com/user-attachments/assets/85c5ec03-633b-43c6-b190-e5a772f7b011" />

---

Go to instance CLI and write;

```
aws eks update-kubeconfig --name CLUSTER NAME --region CLUSTER REGION 
aws eks update-kubeconfig --name EKS_CLOUD --region ap-south-1
 
```

Let’s see the nodes

```
kubectl get nodes
```
Now Give this command in CLI

< cat /root/.kube/config >

Copy the config file to Jenkins master or the local file manager and save it

copy it and save it in documents or another folder save it as secret-file.txt

Install Kubernetes Plugin, Once it’s installed successfully

<img width="757" height="331" alt="image" src="https://github.com/user-attachments/assets/a7eefca3-7d11-4990-a755-2cb5785567fe" />

---

goto manage Jenkins –> manage credentials –> Click on Jenkins global –> add credentials

<img width="872" height="392" alt="image" src="https://github.com/user-attachments/assets/95d5aa67-29b9-4b08-88f0-b9c52914f4b5" />

---

final step to deploy on the Kubernetes cluster

```groovy
stage('Deploy to Kubernetes') {
    steps {
        script {
            dir('K8S') {
                withKubeConfig(credentialsId: 'k8s') {
                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                }
            }
        }
    }
}
```

Verify:

```bash
kubectl get all
```

Copy **External IP** → Open in browser.

---
# 🔥 Output Screenshot (Final UI)

Open:

```
http://<EXTERNAL-IP>:3000
```
<img width="889" height="384" alt="image" src="https://github.com/user-attachments/assets/37db55cd-0c8d-4b3c-a9e2-04bd1e9d0c1e" />

---

# 🧨 Step 4 — Cleanup (Destroy Everything)

Now Go to Jenkins Dashboard and click on Terraform-Eks job

And build with parameters and destroy action

It will delete the EKS cluster that provisioned

<img width="660" height="197" alt="image" src="https://github.com/user-attachments/assets/79e9fd33-4b78-496f-bb57-3703c7472310" />

After 10 minutes cluster will delete and wait for it. Don’t remove ec2 instance till that time.

<img width="711" height="243" alt="image" src="https://github.com/user-attachments/assets/c86562f9-f93c-4d28-b572-d41fa0c329fd" />

Cluster deleted

<img width="715" height="188" alt="image" src="https://github.com/user-attachments/assets/592ec9a3-0bf6-41b6-848a-520c7524b7f9" />

Delete the Ec2 instance & IAM role.

Check the load balancer also if it is deleted or not.

Congratulations on completing the journey of deploying your Hotstar clone using DevSecOps practices on AWS! This process has highlighted the power of integrating security measures seamlessly into the deployment pipeline, ensuring not only efficiency but also a robust shield against potential threats.

### Key Highlights:

  - Leveraging AWS services, Docker, Jenkins, and security tools, we orchestrated a secure and automated deployment pipeline.
  - Implementing DevSecOps principles helped fortify the application against vulnerabilities through continuous security checks.
  - The seamless integration of static code analysis, container security, and automated deployment showcases the strength of DevSecOps methodologies.

---

---

# 🔓 Ports Used

| Service      | Port            |
| ------------ | --------------- |
| Jenkins      | 8080            |
| SonarQube    | 9000            |
| Hotstar App  | 3000            |
| Docker       | Default         |
| EKS Services | 3000, NodePorts |

---
























































































































































































































