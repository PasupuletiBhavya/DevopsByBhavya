# 🚀 End-to-End CI/CD Pipeline with Jenkins on AWS
By Bhavya Pasupuleti • DevOps Engineer • 2025

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png)

## 📌 Introduction
In today’s fast-paced software development world, DevOps plays a critical role in automating the build, test, and deployment process. In this blog, I’ll walk you through a complete **end-to-end CI/CD pipeline** I built using **Jenkins on AWS EC2**, integrating popular DevOps tools like **Git, Maven, SonarQube, Nexus**, and **Tomcat**.

This hands-on project gave me real-world experience in deploying a robust and scalable automation pipeline.

---

## 🛠️ Tech Stack

- Git: Source code management  
- Maven: Build tool  
- SonarQube: Code quality scanner  
- Nexus: Artifact repository  
- Tomcat: Web server  
- Jenkins: CI/CD automation  
- AWS EC2: Infrastructure

---

## ⚙️ Step-by-Step Process

### ✅ STEP 1: Launch EC2 Instances

| Service   | Instance Type | Volume     |
|-----------|---------------|------------|
| Jenkins   | t2.micro      | Default    |
| Tomcat    | t2.micro      | Default    |
| SonarQube | t2.medium     | 25GB EBS   |
| Nexus     | t2.medium     | 25GB EBS   |

Make sure all are in the same security group and PEM key.

---

### ✅ STEP 2: Install Tools

---

### 📦 Jenkins Setup

```bash
# Install Git
yum install git -y

# Add Jenkins repo
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade

# Install dependencies
sudo yum install fontconfig java-21-openjdk -y
sudo yum install jenkins -y

# Install Java 17 (optional override)
yum install java-17-amazon-corretto -y

# Start Jenkins
systemctl daemon-reload
systemctl start jenkins.service
systemctl status jenkins.service

# Install Java
yum install java-17-amazon-corretto -y

# Download & extract Tomcat
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
tar -zxvf apache-tomcat-9.0.104.tar.gz

# Configure GUI & Script Access
sed -i '56  a\<role rolename="manager-gui"/>' conf/tomcat-users.xml
sed -i '57  a\<role rolename="manager-script"/>' conf/tomcat-users.xml
sed -i '58  a\<user username="tomcat" password="admin@123" roles="manager-gui, manager-script"/>' conf/tomcat-users.xml
sed -i '59  a\</tomcat-users>' conf/tomcat-users.xml
sed -i '56d' conf/tomcat-users.xml

# Remove IP restrictions
sed -i '21d' webapps/manager/META-INF/context.xml
sed -i '22d' webapps/manager/META-INF/context.xml

# Start Tomcat
sh bin/startup.sh

---

### 📦 Nexus Setup

```bash
# Install Java
yum install java-17-amazon-corretto -y

# Download Nexus
cd /opt
wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
tar -zxvf nexus-unix-x86-64-3.79.0-09.tar.gz

# Create Nexus user
useradd nexus

# Edit nexus script to run as nexus user
vim /opt/nexus-3.79.0-09/bin/nexus
# → set: run_as_user="nexus"

# Start Nexus
su - nexus
/opt/nexus-3.79.0-09/bin/nexus start
/opt/nexus-3.79.0-09/bin/nexus status

# Check ports and default admin password
ps -ef | grep nexus
ss -tuln | grep 8081
cat /opt/sonatype-work/nexus3/admin.password

# Install Java 11
amazon-linux-extras install java-openjdk11 -y

# Download SonarQube
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip

# Unzip & set permissions
unzip sonarqube-8.9.6.50800.zip
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R

# Start SonarQube
su - sonar
cd /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/
sh sonar.sh start
sh sonar.sh status

✅ STEP 3: Jenkins Plugin Installation
Go to http://<Jenkins-IP>:8080

Login using the initial admin password (found in /var/lib/jenkins/secrets/initialAdminPassword)

Navigate to Manage Jenkins → Plugin Manager

Install the following plugins:

SonarQube Scanner

Nexus Artifact Uploader

SSH Agent

---

## 🛠️ Jenkins Pipeline Script (Scripted Pipeline)

```groovy
node {

  stage("Code") {
    git "https://github.com/PasupuletiBhavya/one.git"
  }

  stage("Build") {
    def mavenHome = tool name: "maven3", type: "maven"
    def mavenCMD = "${mavenHome}/bin/mvn"
    sh "${mavenCMD} clean package"
  }

  stage("CQA") {
    withSonarQubeEnv('mysonar') {
      def mavenHome = tool name: "maven3", type: "maven"
      def mavenCMD = "${mavenHome}/bin/mvn"
      sh "${mavenCMD} sonar:sonar"
    }
  }

  stage("Nexus") {
    nexusArtifactUploader artifacts: [[
      artifactId: 'myweb',
      classifier: '',
      file: 'target/myweb-8.6.9.war',
      type: 'war'
    ]],
    credentialsId: 'nexus',
    groupId: 'in.javahome',
    nexusUrl: '34.229.123.124:8081',
    nexusVersion: 'nexus3',
    protocol: 'http',
    repository: 'bhavya-repo',
    version: '8.6.9'
  }

  stage("Deployment") {
    sshagent(['8bce0b0e-c5db-4aac-bf07-4831ca13a760']) {
      sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@52.23.240.218:/home/ec2-user/apache-tomcat-9.0.104/webapps/'
    }
  }

}

Make sure these tools are configured:

Maven: Manage Jenkins → Global Tool Configuration

Sonar Token: My Account → Security → Generate Token

SSH Key: Added in Jenkins → Manage Credentials

📁 After every stage:

✅ Check for .war file in target/ folder after build

✅ Verify SonarQube dashboard for bugs and vulnerabilities

✅ See WAR file in Nexus hosted repo

✅ Open Tomcat server → Click WAR to launch the app

<iframe src="https://drive.google.com/file/d/1VXveO2Sa9ZyhlAniLNATR3eiGrEabdZg/preview" width="640" height="480"></iframe>
