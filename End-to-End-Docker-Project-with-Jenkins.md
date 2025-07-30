# 🚀 End-to-End Docker Project with Jenkins CI/CD | Deploying a Node.js App with Security Scans

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1747971726993/06a792a9-6082-40a8-82de-2ad051da2a0e.png)

## 📌 Introduction

This blog walks through a full DevOps pipeline for deploying a Node.js application using Jenkins, Docker, Trivy, SonarQube, and OWASP Dependency-Check. From code checkout to vulnerability scanning and deployment, everything is automated.

You’ll learn how to:

* Clone code from GitHub and scan it with SonarQube
* Run OWASP and Trivy scans for security
* Build Docker images and push to Docker Hub
* Deploy the containerized Node.js app

A perfect portfolio project for any aspiring DevOps engineer!

---

## 🛠️ Tech Stack

* **Jenkins**: CI/CD automation
* **Git**: Source control
* **Node.js**: Application framework
* **SonarQube**: Code quality analysis
* **OWASP Dependency-Check**: Vulnerability scanning
* **Trivy**: Docker image scanning
* **Docker**: Containerization
* **AWS EC2**: Infrastructure

---

## 🚀 Step-by-Step Guide

### 🔹 Step 1: Launch EC2 Instance

* Instance Type: `t2.large`
* OS: Amazon Linux 2
* Ports: Open port `22` (SSH)

### 🔹 Step 2: Install Jenkins, Git, Docker, and Trivy

Create and run a shell script (`jenkins.sh`):

```bash
# Install Git
yum install git -y

# Jenkins repo and install
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum upgrade -y
yum install fontconfig java-21-openjdk jenkins -y

# Start Jenkins
systemctl daemon-reload
systemctl start jenkins
systemctl status jenkins

# Install Docker
yum install docker -y
systemctl start docker
chmod 777 /var/run/docker.sock
```

Install **Trivy**:

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
mv trivy /usr/local/bin/
```

Fix PATH:

```bash
echo 'export PATH=$PATH:/usr/local/bin/' >> ~/.bashrc
source ~/.bashrc
```

---

### 🔹 Step 3: Install Jenkins Plugins

Install the following plugins:

* SonarQube Scanner
* NodeJS
* OWASP Dependency-Check
* Docker Pipeline
* AdoptOpenJDK (Temurin)

Via: `Manage Jenkins → Plugin Manager → Available` → search → install.

---

### 🔹 Step 4: Run SonarQube in Docker

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Login to `http://<ec2-ip>:9000` with `admin`/`admin`. Create a dummy project to test access.

---

### 🔹 Step 5: Configure Jenkins Credentials & SonarQube

1. Generate a SonarQube token under: `My Account → Security`
2. Add to Jenkins as **Secret text** with ID `sonar`
3. Configure Jenkins > SonarQube Server:

   * Name: `mysonar`
   * URL: `http://<ec2-ip>:9000`
   * Auth Token: use the credential created

Add a webhook in SonarQube:

* Go to: `Administration → Configuration → Webhooks`
* URL: `http://<jenkins-ip>:8080/sonarqube-webhook/`

---

### 🔹 Step 6: Declarative Jenkins Pipeline

```groovy
pipeline {
  agent any

  tools {
    jdk 'jdk17'
    nodejs 'node16'
  }

  environment {
    SCANNER_HOME = tool 'mysonar'
  }

  stages {
    stage('Clean') {
      steps { cleanWs() }
    }

    stage('Checkout Code') {
      steps {
        git 'https://github.com/PasupuletiBhavya/Zomato-Project.git'
      }
    }

    stage('Sonar Scan') {
      steps {
        withSonarQubeEnv('mysonar') {
          sh '''
            $SCANNER_HOME/bin/sonar-scanner \
            -Dsonar.projectName=zomato \
            -Dsonar.projectKey=zomato
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
      }
    }

    stage('Install Dependencies') {
      steps { sh 'npm install' }
    }

    stage('OWASP Scan') {
      steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dp-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }

    stage('Trivy File System Scan') {
      steps {
        sh 'trivy fs . > trivyfs.txt'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t image1 .'
      }
    }

    stage('Docker Push') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker-password') {
            sh 'docker tag image1 bhavyap007/dockerproject:zomato'
            sh 'docker push bhavyap007/dockerproject:zomato'
          }
        }
      }
    }

    stage('Trivy Image Scan') {
      steps {
        sh 'trivy image bhavyap007/dockerproject:zomato'
      }
    }

    stage('Deploy App') {
      steps {
        sh 'docker run -d --name zomato -p 3000:3000 bhavyap007/dockerproject:zomato'
      }
    }
  }
}
```

---

## ✅ Final Pipeline Flow

1. Launch EC2 and install Jenkins, Docker, Git, etc.
2. Install and configure Trivy, SonarQube, and OWASP
3. Set up Jenkins credentials and tools
4. Clone project and run SonarQube scan
5. Run quality gates, install npm packages
6. Perform security scans
7. Build and push Docker image
8. Scan image and deploy container

---

## 🧠 What I Learned

* Setting up CI/CD for Node.js with Docker
* Code and image vulnerability scanning
* SonarQube integration and quality gates
* Jenkins plugins, credentials, and environment config
* Writing secure, declarative Jenkins pipelines

---

## 🔗 Resources

* 🔗 [GitHub – PasupuletiBhavya](https://github.com/PasupuletiBhavya)
* 🌐 [Jenkins](https://www.jenkins.io/)
* 🐳 [Docker](https://www.docker.com/)
* 📦 [SonarQube](https://www.sonarsource.com/products/sonarqube/)
* 🔍 [Trivy](https://aquasecurity.github.io/trivy/)
* 🛡️ [OWASP Dependency-Check](https://jeremylong.github.io/DependencyCheck/)

---

📺 **Video Demo**:
[Watch on Google Drive](https://drive.google.com/file/d/1SfiD_lUYvSbhqlNPDhcmtxNvVZVKURly/preview)

📖 **Read Full Blog on Hashnode**: [https://devops-by-bhavya.hashnode.dev/](https://devops-by-bhavya.hashnode.dev/)

🙋‍♀️ If this blog helped you, feel free to connect on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/) or star the repo on GitHub. Let's grow together!

