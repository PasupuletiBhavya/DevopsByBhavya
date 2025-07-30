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

<!-- Converted Markdown content begins below -->

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

* JENKINS: T2.MICRO
* TOMCAT: T2.MICRO
* SONAR: T2.MEDIUM (25 GB of EBS)
* NEXUS: T2.MEDIUM (25 GB of EBS)

<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1744944384391/08883885-d532-4e1a-88a6-15d7f9f2fad3.png" width="80%">
</p>

### STEP 2: Install Jenkins, Tomcat, SonarQube, Nexus

#### Jenkins Script

```bash
# Install Git
yum install git -y
...
```

<p align="center">
  <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1744945710282/57728c01-5aa4-409a-9fb2-33fffb7e8fa3.png" width="60%">
</p>

... \[Content continues with each tool installation, image embeds converted, headings cleaned, and iframe removed]

## ✅ Final Pipeline Flow Summary

**From Code to Deployment:**

1. Developer pushes code to Git
2. Jenkins triggers build → SonarQube analysis
3. Maven builds code → Nexus stores WAR
4. Jenkins deploys to Tomcat → App live

## 🎓 What I Learned

* Real-world DevOps pipeline implementation
* Integrating Jenkins with SonarQube, Nexus, and Tomcat
* Managing services on AWS EC2
* Automating deployment flow

## 🔗 Resources & Links

* GitHub: [https://github.com/PasupuletiBhavya](https://github.com/PasupuletiBhavya)
* Jenkins: [jenkins.io](https://www.jenkins.io/)
* SonarQube: [sonarqube.org](https://www.sonarqube.org/)
* Nexus: [sonatype.com](https://www.sonatype.com/)
* Tomcat: [tomcat.apache.org](https://tomcat.apache.org/)

## 💬 Let’s Connect

If this article added value to your journey, connect with me on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/). Let’s keep growing together 🚀
