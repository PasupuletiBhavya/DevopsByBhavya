---

title: "End-to-End CI/CD Pipeline with Jenkins on AWS"
seoTitle: "Jenkins CI/CD Pipeline on AWS"
seoDescription: "Learn how to set up a real-world CI/CD pipeline using Jenkins, Maven, SonarQube, Nexus, and Tomcat on AWS EC2. Includes video demo and deployment."
datePublished: Fri Apr 18 2025 06:36:56 GMT+0000 (Coordinated Universal Time)
cuid: cm9mf2gt4000g09jrgzpr3mj9
slug: end-to-end-cicd-pipeline-with-jenkins-on-aws
cover: [https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png)
ogImage: [https://cdn.hashnode.com/res/hashnode/image/upload/v1744958202876/ff61b9a9-5d45-4c63-b8aa-cc2d10c37af1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1744958202876/ff61b9a9-5d45-4c63-b8aa-cc2d10c37af1.png)
tags: aws, github, sonarqube, maven, devops, jenkins, tomcat, pipeline, nexus, ci-cd, codenewbies, gitcommands, devops-cicd-jenkins-aws-maven-sonarqube-nexus-tomcat
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h1 align="center">🚀 End-to-End CI/CD Pipeline with Jenkins on AWS</h1>
<p align="center"><i>By Bhavya Pasupuleti • DevOps Engineer • 2025</i></p>

<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png" width="80%">
</p>

## Introduction

In today’s fast-paced software development world, DevOps plays a critical role in automating the build, test, and deployment process. In this blog, I’ll walk you through a complete **end-to-end CI/CD pipeline** I built using **Jenkins on AWS EC2**, integrating popular DevOps tools like **Git, Maven, SonarQube, Nexus**, and **Tomcat**. This hands-on project not only helped me understand how each tool fits into the software development lifecycle but also gave me real-world experience in deploying a robust and scalable automation pipeline.

## 🔧 Tech Stack

* Git: Source code management
* Maven: Build tool
* SonarQube: Code quality check
* Nexus: Artifact storage
* Tomcat: Webserver for deployment
* Jenkins: CI/CD pipeline
* AWS EC2: Hosting all services

## ⚙️ Step-by-Step Process to Deploy the Application

### STEP 1: Launch 4 EC2 Instances with Same PEM File

* Jenkins: t2.micro
* Tomcat: t2.micro
* SonarQube: t2.medium (25 GB of EBS)
* Nexus: t2.medium (25 GB of EBS)

<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1744944384391/08883885-d532-4e1a-88a6-15d7f9f2fad3.png" width="80%">
</p>

### STEP 2: Install Jenkins, Tomcat, SonarQube, Nexus

Each instance was configured with its respective service using shell scripts. Here are brief examples:

#### Jenkins

```bash
# Install Git and Java
sudo yum install git java-17-amazon-corretto -y
# Add Jenkins repo and install
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl start jenkins
```

#### Tomcat

```bash
# Install Java
sudo yum install java-17-amazon-corretto -y
# Download and extract Tomcat
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
# Configure users and remove IP restrictions
# Start Tomcat
sh apache-tomcat-9.0.104/bin/startup.sh
```

#### SonarQube

```bash
# Install Java OpenJDK 11
amazon-linux-extras install java-openjdk11 -y
# Download and unzip SonarQube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
# Create sonar user and start service
```

#### Nexus

```bash
# Install Java
yum install java-17-amazon-corretto -y
# Download and extract Nexus
wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
# Start Nexus and retrieve admin password
```

## 🧪 Jenkins Pipeline Setup

Once Jenkins and all other services were up, I created a **Scripted Pipeline Job** in Jenkins with the following stages:

### STAGE 1: Get Code from GitHub

```groovy
node {
  stage("Code") {
    git 'https://github.com/PasupuletiBhavya/one.git'
  }
```

### STAGE 2: Build the Source Code

```groovy
  stage("Build") {
    def mavenHome = tool name: "maven3", type: "maven"
    sh "${mavenHome}/bin/mvn clean package"
  }
```

### STAGE 3: SonarQube Analysis

```groovy
  stage("CQA") {
    withSonarQubeEnv('mysonar') {
      sh "${mavenHome}/bin/mvn sonar:sonar"
    }
  }
```

### STAGE 4: Upload Artifact to Nexus

```groovy
  stage("Nexus") {
    nexusArtifactUploader artifacts: [[
      artifactId: 'myweb', file: 'target/myweb-8.6.9.war', type: 'war'
    ]],
    credentialsId: 'nexus',
    groupId: 'in.javahome',
    nexusUrl: 'http://<nexus-ip>:8081',
    nexusVersion: 'nexus3',
    repository: 'bhavya-repo',
    version: '8.6.9'
  }
```

### STAGE 5: Deploy to Tomcat

```groovy
  stage("Deployment") {
    sshagent(['ssh-key-id']) {
      sh 'scp target/*.war ec2-user@<tomcat-ip>:/home/ec2-user/apache-tomcat-9.0.104/webapps/'
    }
  }
}
```

<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1744952395185/95a0f441-2324-45b1-b586-6d86d32cce50.png" width="80%">
</p>

## ✅ Final Pipeline Flow Summary

1. Code pushed to GitHub
2. Jenkins job pulls code
3. Maven builds WAR
4. SonarQube scans it
5. WAR uploaded to Nexus
6. WAR deployed to Tomcat

## 🎓 What I Learned

* How CI/CD pipelines work in real-world projects
* Jenkins integration with Maven, SonarQube, Nexus
* How to manage multiple EC2 instances efficiently
* Automating deployments end-to-end

## 🔗 Resources & Links

* GitHub: [https://github.com/PasupuletiBhavya](https://github.com/PasupuletiBhavya)
* Jenkins: [jenkins.io](https://www.jenkins.io/)
* SonarQube: [sonarqube.org](https://www.sonarqube.org/)
* Nexus: [sonatype.com](https://www.sonatype.com/)
* Tomcat: [tomcat.apache.org](https://tomcat.apache.org/)

## 💬 Let’s Connect

If this article added value to your journey, connect with me on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/). Let’s keep growing together 🚀
