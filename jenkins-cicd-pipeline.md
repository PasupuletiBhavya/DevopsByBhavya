# 🚀 End-to-End CI/CD Pipeline with Jenkins on AWS

*By Bhavya Pasupuleti • DevOps Engineer • 2025*

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png)

---

## 📌 Introduction

In today’s fast-paced software development world, DevOps plays a critical role in automating the build, test, and deployment process. In this blog, I’ll walk you through a complete **end-to-end CI/CD pipeline** I built using **Jenkins on AWS EC2**, integrating popular DevOps tools like **Git, Maven, SonarQube, Nexus**, and **Tomcat**.

---

## 🛠️ Tech Stack

- Git
- Maven
- SonarQube
- Nexus
- Tomcat
- Jenkins
- AWS EC2

---

## ⚙️ Step-by-Step Process

### Step 1: Launch EC2 Instances

- Jenkins: `t2.micro`
- Tomcat: `t2.micro`
- SonarQube: `t2.medium` (25GB EBS)
- Nexus: `t2.medium` (25GB EBS)

![EC2 Setup](https://cdn.hashnode.com/res/hashnode/image/upload/v1744944384391/08883885-d532-4e1a-88a6-15d7f9f2fad3.png)

---

### Step 2: Install Tools

#### ✅ Jenkins Setup

```bash
yum install git -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo yum install fontconfig java-21-openjdk jenkins
sudo systemctl daemon-reload
yum install java-17-amazon-corretto -y
systemctl start jenkins.service
```

![Jenkins Setup](https://cdn.hashnode.com/res/hashnode/image/upload/v1744945710282/57728c01-5aa4-409a-9fb2-33fffb7e8fa3.png)

---

#### ✅ Tomcat Setup

```bash
yum install java-17-amazon-corretto -y
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
tar -zxvf apache-tomcat-9.0.104.tar.gz
# Configure users and IP restrictions
sh bin/startup.sh
```

![Tomcat UI](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946107499/47ee20b5-8821-49f2-8314-02216df4d61b.png)

---

#### ✅ Nexus Setup

```bash
yum install java-17-amazon-corretto -y
cd /opt
wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
tar -zxvf nexus-unix-x86-64-3.79.0-09.tar.gz
useradd nexus
# edit run_as_user="nexus"
su - nexus
/opt/nexus-3.79.0-09/bin/nexus start
```

![Nexus Login](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946970500/33385c8b-4911-43d8-8aef-d767a170d362.png)

---

#### ✅ SonarQube Setup

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
```

---

### Step 3: Configure Jenkins Pipeline

1. Install required plugins:
   - SonarQube Scanner
   - Nexus Artifact Uploader
   - SSH Agent

2. Create a new pipeline job and use this Jenkinsfile:

```groovy
node {
  stage("Code") {
    git "https://github.com/PasupuletiBhavya/one.git"
  }

  stage("Build") {
    def mavenHome = tool name: "maven3", type: "maven"
    sh "${mavenHome}/bin/mvn clean package"
  }

  stage("CQA") {
    withSonarQubeEnv('mysonar') {
      def mavenHome = tool name: "maven3", type: "maven"
      sh "${mavenHome}/bin/mvn sonar:sonar"
    }
  }

  stage("Nexus") {
    nexusArtifactUploader artifacts: [[
        artifactId: 'myweb',
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
```

---

## 🎥 Video Demo

📺 [Watch the Deployment Video](https://drive.google.com/file/d/1VXveO2Sa9ZyhlAniLNATR3eiGrEabdZg/view)

---

## ✅ Final Pipeline Summary

1. Developer pushes code to GitHub  
2. Jenkins triggers → SonarQube analysis  
3. Maven builds → Nexus stores WAR  
4. Jenkins deploys WAR to Tomcat

---

## 📘 What I Learned

- How DevOps tools work together in CI/CD
- Jenkins pipeline writing
- Automating builds, code scanning, artifact management
- Deploying apps end-to-end on AWS EC2

---

## 🔗 Resources

- [GitHub](https://github.com/PasupuletiBhavya)
- [Jenkins](https://www.jenkins.io/)
- [SonarQube](https://www.sonarqube.org/)
- [Nexus](https://www.sonatype.com/)
- [Apache Tomcat](https://tomcat.apache.org/)

---

💬 Let’s connect on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/) for more DevOps content and collaboration opportunities!
