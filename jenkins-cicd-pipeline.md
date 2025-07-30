
# 🚀 End-to-End CI/CD Pipeline with Jenkins on AWS

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png)

## 📌 Introduction

In today’s fast-paced software development world, DevOps plays a critical role in automating the build, test, and deployment process. In this blog, I’ll walk you through a complete **end-to-end CI/CD pipeline** I built using **Jenkins on AWS EC2**, integrating popular DevOps tools like **Git, Maven, SonarQube, Nexus**, and **Tomcat**.

This hands-on project not only helped me understand how each tool fits into the software development lifecycle but also gave me real-world experience in deploying a robust and scalable automation pipeline.

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

## 🧭 Step-by-Step Deployment Guide

### 🔹 STEP 1: Launch 4 EC2 Instances with the Same PEM File

- Jenkins: t2.micro  
- Tomcat: t2.micro  
- SonarQube: t2.medium (25 GB EBS)  
- Nexus: t2.medium (25 GB EBS)

![EC2 Setup](https://cdn.hashnode.com/res/hashnode/image/upload/v1744944384391/08883885-d532-4e1a-88a6-15d7f9f2fad3.png)

---

### 🔹 STEP 2: Install Services on EC2

#### Jenkins Script

```bash
yum install git -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo yum install fontconfig java-21-openjdk
sudo yum install jenkins
systemctl daemon-reload
yum install java-17-amazon-corretto -y
yum install jenkins -y
systemctl start jenkins
systemctl status jenkins
```

![Jenkins Status](https://cdn.hashnode.com/res/hashnode/image/upload/v1744945710282/57728c01-5aa4-409a-9fb2-33fffb7e8fa3.png)

---

#### Tomcat Script

```bash
yum install java-17-amazon-corretto -y
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
tar -zxvf apache-tomcat-9.0.104.tar.gz
cd apache-tomcat-9.0.104
sed -i '56  a<role rolename="manager-gui"/>' conf/tomcat-users.xml
sed -i '57  a<role rolename="manager-script"/>' conf/tomcat-users.xml
sed -i '58  a<user username="tomcat" password="admin@123" roles="manager-gui, manager-script"/>' conf/tomcat-users.xml
sed -i '59  a</tomcat-users>' conf/tomcat-users.xml
sed -i '56d' conf/tomcat-users.xml
sed -i '21d' webapps/manager/META-INF/context.xml
sed -i '22d' webapps/manager/META-INF/context.xml
sh bin/startup.sh
```

![Tomcat Running](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946194488/39623112-6f75-4b76-a7d0-22cfc6527b1a.png)

---

#### Nexus Script

```bash
yum install java-17-amazon-corretto -y
cd /opt
wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
tar -zxvf nexus-unix-x86-64-3.79.0-09.tar.gz
useradd nexus
vim /opt/nexus-3.79.0-09/bin/nexus  # Edit: run_as_user="nexus"
su - nexus
/opt/nexus-3.79.0-09/bin/nexus start
/opt/nexus-3.79.0-09/bin/nexus status
cat /opt/sonatype-work/nexus3/admin.password
```

![Nexus Running](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946970500/33385c8b-4911-43d8-8aef-d767a170d362.png)

---

#### SonarQube Script

```bash
amazon-linux-extras install java-openjdk11 -y
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
su - sonar
cd /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/
sh sonar.sh start
sh sonar.sh status
```

---

### 🔹 STEP 3: Jenkins Plugin Installation

- Go to `http://<your-ip>:8080`  
- Navigate to **Manage Jenkins → Plugin Manager → Available**  
- Install:
  - **SonarQube Scanner**
  - **Nexus Artifact Uploader**
  - **SSH Agent**

---

### 🔹 STEP 4: Scripted Jenkins Pipeline

#### STAGE 1: Pull Code

```groovy
stage("Code") {
  git "https://github.com/PasupuletiBhavya/one.git"
}
```

#### STAGE 2: Build with Maven

```groovy
stage("Build") {
  def mavenHome = tool name: "maven3"
  def mavenCMD = "${mavenHome}/bin/mvn"
  sh "${mavenCMD} clean package"
}
```

#### STAGE 3: SonarQube Scan

```groovy
stage("CQA") {
  withSonarQubeEnv('mysonar') {
    def mavenHome = tool name: "maven3"
    def mavenCMD = "${mavenHome}/bin/mvn"
    sh "${mavenCMD} sonar:sonar"
  }
}
```

#### STAGE 4: Upload WAR to Nexus

```groovy
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
```

#### STAGE 5: Deploy to Tomcat

```groovy
stage("Deployment") {
  sshagent(['8bce0b0e-c5db-4aac-bf07-4831ca13a760']) {
    sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@52.23.240.218:/home/ec2-user/apache-tomcat-9.0.104/webapps/'
  }
}
```

---

## 🔄 Final CI/CD Flow Summary

1. Developer pushes code to GitHub  
2. Jenkins job triggers → SonarQube analysis  
3. Maven builds code → Nexus stores WAR file  
4. Jenkins deploys to Tomcat → Application is live

---

## 🧠 What I Learned

- Real-time integration of core DevOps tools
- Jenkins pipeline structure and plugin usage
- Maven project structure and SonarQube integration
- WAR file deployment automation with Tomcat

---

## 🎥 Video Demo

> [Click to watch the video](https://drive.google.com/file/d/1VXveO2Sa9ZyhlAniLNATR3eiGrEabdZg/preview)

---

## 🔗 Useful Links

- [GitHub](https://github.com/PasupuletiBhavya)  
- [Jenkins](https://www.jenkins.io/)  
- [SonarQube](https://www.sonarqube.org/)  
- [Nexus](https://www.sonatype.com/)  
- [Apache Tomcat](https://tomcat.apache.org/)

---

📖 See Full Blog with Images & Demo: https://devops-by-bhavya.hashnode.dev/

**Connect with me on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/)** if you have questions or want to collaborate! 💬✨

