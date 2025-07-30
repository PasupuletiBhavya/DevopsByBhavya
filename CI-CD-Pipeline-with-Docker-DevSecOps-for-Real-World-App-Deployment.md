# 🚀 CI/CD Pipeline with Docker: DevSecOps for Real-World App Deployment

> A practical DevSecOps pipeline built entirely with Docker, Jenkins, and Terraform — from code to cloud.

📖 **[Full 6-phase DevSecOps series](https://devops-by-bhavya.hashnode.dev/campus-drive-management-a-complete-devsecops-journey-6-phases-explained)**

---

## 📚 Table of Contents

- [🧠 Project Overview](#-project-overview)
- [🏕️ Yelp Camp Architecture](#-yelp-camp-architecture)
- [🗂️ Database Strategy](#-database-strategy)
- [🔐 Environment Variables](#-environment-variables)
- [📸 Image Upload (Cloudinary)](#-image-upload-cloudinary)
- [🗺️ Location Mapping (Mapbox)](#-location-mapping-mapbox)
- [🛢️ MongoDB Atlas Setup](#-mongodb-atlas-setup)
- [🖥️ EC2 Instance Setup](#-ec2-instance-setup)
- [🐳 Dockerizing the Application](#-dockerizing-the-application)
- [⚙️ Terraform EC2 Provisioning](#-terraform-ec2-provisioning)
- [🤝 Jenkins Master-Slave Setup](#-jenkins-master-slave-setup)
- [📦 Jenkins Pipeline (Dev)](#-jenkins-pipeline-dev)
- [🧪 Jenkins Pipeline (Test)](#-jenkins-pipeline-test)
- [📢 Slack Notifications](#-slack-notifications)
- [📊 Final Pipeline Flow Summary](#-final-pipeline-flow-summary)
- [💡 What I Learned](#-what-i-learned)

---

## 🧠 Project Overview

Built an end-to-end **CI/CD pipeline** for the `Yelp Camp` web application using:

- **Docker** (Containerization)
- **Jenkins** (CI/CD Automation)
- **Trivy** (Security Scanning)
- **Terraform** (Infrastructure as Code)
- **SonarQube** (Code Quality)
- **Slack** (Notification)
- **MongoDB Atlas, Cloudinary, Mapbox**

---

## 🏕️ Yelp Camp Architecture

### 🔧 Three-Tier Web App

1. **Frontend**: UI for interacting with the campgrounds  
2. **Backend**: Node.js server for routing, auth, business logic  
3. **Database**: MongoDB Atlas to store all data  

---

## 🗂️ Database Strategy

| Option | Approach | Verdict |
|--------|----------|---------|
| 🧪 Option 1 | MongoDB in K8s | ❌ Manual setup, backup risk |
| ✅ Option 2 | MongoDB Atlas | ✔️ Fully managed, cloud-hosted |

---

## 🔐 Environment Variables

```.env
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_KEY=
CLOUDINARY_SECRET=
MAPBOX_TOKEN=
DB_URL=
SECRET=my_secret_key
```

---

## 📸 Image Upload (Cloudinary)

- Go to [cloudinary.com](https://cloudinary.com)
- Copy `Cloud name`, `API Key`, `API Secret`
- Add them to `.env`

---

## 🗺️ Location Mapping (Mapbox)

- Go to [Mapbox Account](https://account.mapbox.com)
- Copy `Default Public Token`
- Add to `.env` as `MAPBOX_TOKEN`

---

## 🛢️ MongoDB Atlas Setup

- Create a **Free Cluster**
- Add DB user (with password)
- Choose “Connect → Drivers → Node.js” to get the `DB_URL`
- Allow IP access from anywhere (0.0.0.0/0)

---

## 🖥️ EC2 Instance Setup

- Launch `t2.large` EC2 (Amazon Linux 2)
- Open ports `22`, `80`, `8080`
- SSH and install:
  - `Docker`, `Git`, `Jenkins`, `Trivy`, `Terraform`, `Java`

---

## 🐳 Dockerizing the Application

### Dockerfile (Node:Alpine)

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD npm start
```

### Commands

```bash
docker build -t appimage .
docker run -itd --name camp-app -p 1111:3000 appimage
```

---

## ⚙️ Terraform EC2 Provisioning

### `provider.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}
```

### `resource.tf`

```hcl
resource "aws_instance" "devserver" {
  ami           = "ami-0e9bbd70d26d7cf4f"
  instance_type = "t2.medium"
  key_name      = "master-slave"
  availability_zone = "us-east-1a"
  ...
}
```

### Commands

```bash
terraform init
terraform apply --auto-approve
```

---

## 🤝 Jenkins Master-Slave Setup

1. Setup SSH agent connection between Jenkins master and EC2 slave
2. Add node label `dev` and `test` for each server
3. Install NodeJS, Docker, Git plugins

---

## 📦 Jenkins Pipeline (Dev)

```groovy
pipeline {
  agent { label 'dev' }
  tools { nodejs 'node16' }
  environment { SCANNER_HOME = tool 'mysonar' }
  stages {
    stage('Code Checkout') {
      steps {
        git "https://github.com/PasupuletiBhavya/devsecops-project.git"
      }
    }
    ...
  }
}
```

---

## 🧪 Jenkins Pipeline (Test)

- Use `terraform workspace new test` to isolate
- Update tag `test-v1`
- Change node label from `dev` to `test`
- Follow similar steps to Dev pipeline

---

## 📢 Slack Notifications

```groovy
post {
  always {
    slackSend(
      channel: 'my-channel',
      message: "*${currentBuild.currentResult}:* Job `${env.JOB_NAME}` #${env.BUILD_NUMBER} 
🔗 ${env.BUILD_URL}"
    )
  }
}
```

---

## 📊 Final Pipeline Flow Summary

✅ GitHub → SonarQube → Docker Build → Trivy Scan → Docker Hub → EC2 Deployment → Slack Notification → Test Pipeline

---

## 💡 What I Learned

- **Alpine Images** reduce Docker size & improve security
- **CI/CD Pipelines** automate secure delivery
- **Trivy** catches vulnerabilities early
- **Terraform** enables reproducible infrastructure
- **Slack** keeps teams informed

---

## 🔗 Connect With Me

[**LinkedIn – Bhavya Pasupuleti**](https://www.linkedin.com/in/bhavya-pasupuleti/)  
[**Full Blog on Hashnode**](https://devops-by-bhavya.hashnode.dev/cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment)
