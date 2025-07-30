---
title: "End-to-End CI/CD Pipeline with Jenkins on AWS"
seoTitle: "Jenkins CI/CD Pipeline on AWS"
seoDescription: "Learn how to set up a real-world CI/CD pipeline using Jenkins, Maven, SonarQube, Nexus, and Tomcat on AWS EC2. Includes video demo and deployment."
datePublished: Fri Apr 18 2025 06:36:56 GMT+0000 (Coordinated Universal Time)
cuid: cm9mf2gt4000g09jrgzpr3mj9
slug: end-to-end-cicd-pipeline-with-jenkins-on-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744939265256/928b20e6-7602-4a5a-aae8-199cf8e51fcb.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1744958202876/ff61b9a9-5d45-4c63-b8aa-cc2d10c37af1.png
tags: aws, github, sonarqube, maven, devops, jenkins, tomcat, pipeline, nexus, ci-cd, codenewbies, gitcommands, devops-cicd-jenkins-aws-maven-sonarqube-nexus-tomcat

---

# Introduction:

In today’s fast-paced software development world, DevOps plays a critical role in automating the build, test, and deployment process. In this blog, I’ll walk you through a complete **end-to-end CI/CD pipeline** I built using **Jenkins on AWS EC2**, integrating popular DevOps tools like **Git, Maven, SonarQube, Nexus**, and **Tomcat**. This hands-on project not only helped me understand how each tool fits into the software development lifecycle but also gave me real-world experience in deploying a robust and scalable automation pipeline. Whether you're a DevOps beginner or someone brushing up on core concepts, this post will give you a practical view of setting up your own pipeline from scratch.

**Tech Stack:**

* Git: Source code management
    
* Maven: Build tool
    
* SonarQube: Code quality check
    
* Nexus: Artifact storage
    
* Tomcat: Webserver for deployment
    
* Jenkins: CI/CD pipeline
    
* AWS EC2: Hosting all services
    
    ## Step-by-Step Process to Deploy the Application:
    
    **STEP-1:** LAUNCH 4 INSTANCES WITH SAME PEM FILE
    
* JENKINS: T2.MICRO
    
* TOMCAT: T2.MICRO
    
* SONAR: T2.MEDIUM (25 GB OF EBS VOLUME)
    
* NEXUS: T2.MEDIUM (25 GB OF EBS VOLUME)
    
* *SETUP SERVICES IN THEIR RESPECTIVE SERVERS*
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744944384391/08883885-d532-4e1a-88a6-15d7f9f2fad3.png align="center")
    
    **STEP-2**: refer the below scripts to install Jenkins, Tomcat ,Sonar and Nexus on respective servers.
    
    **Jenkins Script**
    
    ```bash
    #STEP-1: INSTALLING GIT
    yum install git -y
    
    #STEP-2: GETTING THE REPO (jenkins.io --> download --> redhat)
    sudo wget -O /etc/yum.repos.d/jenkins.repo \
        https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    sudo yum upgrade
    # Add required dependencies for the jenkins package
    sudo yum install fontconfig java-21-openjdk
    sudo yum install jenkins
    sudo systemctl daemon-reload
    
    #STEP-3: DOWNLOAD JAVA17 AND JENKINS
    yum install java-17-amazon-corretto -y
    yum install jenkins -y
    
    #STEP-4: RESTARTING JENKINS
    systemctl start jenkins.service
    systemctl status jenkins.service
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744945710282/57728c01-5aa4-409a-9fb2-33fffb7e8fa3.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744945897839/5bccfe19-fce6-4ea1-b776-4045f88588cf.png align="center")
    
    **Tomcat Script:**
    
    ```bash
    #STEP-1: INSTALL JAVA 17
    yum install java-17-amazon-corretto -y
    
    #STEP-2: DOWNLOAD & EXTRACT TOMCAT
    wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
    tar -zxvf apache-tomcat-9.0.104.tar.gz
    
    #STEP-3: CONFIGURE TOMCAT USERS (GUI & SCRIPT ACCESS)
    sed -i '56  a\<role rolename="manager-gui"/>' conf/tomcat-users.xml
    sed -i '57  a\<role rolename="manager-script"/>' conf/tomcat-users.xml
    sed -i '58  a\<user username="tomcat" password="admin@123" roles="manager-gui, manager-script"/>' conf/tomcat-users.xml
    sed -i '59  a\</tomcat-users>' conf/tomcat-users.xml
    sed -i '56d' conf/tomcat-users.xml
    #STEP-4: REMOVE IP RESTRICTIONS FOR MANAGER ACCESS
    sed -i '21d' webapps/manager/META-INF/context.xml
    sed -i '22d' webapps/manager/META-INF/context.xml
    
    #STEP-5: START TOMCAT SERVER
    sh bin/startup.sh
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946107499/47ee20b5-8821-49f2-8314-02216df4d61b.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946194488/39623112-6f75-4b76-a7d0-22cfc6527b1a.png align="center")
    
    **Nexus Script:**
    
    ```bash
    #STEP-1: INSTALL JAVA 17
    yum install java-17-amazon-corretto -y
    
    #STEP-2: DOWNLOAD NEXUS AND EXTRACT
    cd /opt
    wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
    tar -zxvf nexus-unix-x86-64-3.79.0-09.tar.gz
    
    #STEP-3: CREATE NEXUS USER & SET PERMISSIONS
    useradd nexus
    vim /opt/nexus-3.79.0-09/bin/nexus
    # (Edit the file and set) run_as_user="nexus"
    
    #STEP-4: SWITCH TO NEXUS USER AND START SERVICE
    su - nexus
    /opt/nexus-3.79.0-09/bin/nexus start
    /opt/nexus-3.79.0-09/bin/nexus status
    
    #STEP-5: VERIFY NEXUS IS RUNNING
    ps -ef | grep nexus
    ss -tuln | grep 8081
    
    #STEP-6: GET DEFAULT ADMIN PASSWORD
    cat /opt/sonatype-work/nexus3/admin.password
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946456175/3926137e-6d33-42e1-8370-3cc6f126eb07.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744946970500/33385c8b-4911-43d8-8aef-d767a170d362.png align="center")
    
    **Sonar Setup:**
    
    ```bash
    #STEP-1: INSTALL JAVA OPENJDK 11
    amazon-linux-extras install java-openjdk11 -y
    
    #STEP-2: DOWNLOAD SONARQUBE
    cd /opt/
    wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
    
    #STEP-3: UNZIP AND CONFIGURE PERMISSIONS
    unzip sonarqube-8.9.6.50800.zip
    useradd sonar
    chown sonar:sonar sonarqube-8.9.6.50800 -R
    
    #STEP-4: SWITCH TO SONAR USER AND START SONARQUBE
    su - sonar
    cd /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/
    sh sonar.sh start
    sh sonar.sh status
    ```
    
    ### STEP-3: Login to Jenkins & Install Plugins
    
    1. Go to `http://<your-ip>:8080` and log in to Jenkins.
        
    2. Go to **Manage Jenkins → Plugin Manager → Available**
        
    3. Install these plugins:
        
        **SonarQube Scanner** – for code scanning
        
        **Nexus Artifact Uploader** – to upload files to Nexus
        
        **SSH Agent** – to send files to another server.
        
    
    **STEP-4**:Create a Jenkins pipeline job and write a Jenkins file for deploy a web application, usually we have 2 types of pipelines,
    
    scripted ,declarative
    
    Here i am using scripted pipeline for the Jenkins file
    
    **STAGE-1 : GET THE CODE FROM GITHUB TO CI-SERVER**  
    node { stage("Code") { git "[https://github.com/PasupuletiBhavya/one.git](https://github.com/PasupuletiBhavya/one.git)" } }  
    **STAGE-2 : BUILD THE SOURCE CODE: GO TO MANAGE JENKINS &gt;&gt; GLOBAL TOOL CONFIGURATION &gt;&gt; MAVEN ADD INSTALLER WITH THE NAME OF maven WITH VERISON (3.9.9)**  
    stage("Build") { def mavenHome = tool name: "maven3", type: "maven" def mavenCMD = "${mavenHome}/bin/mvn" sh "${mavenCMD} clean package" }  
    *CHECK THE WAR FILE IN JENKINS FOR CONFIRMATION AFTER SUCCESSFULL BUILD*
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744950323138/52e103a8-f4a9-47e0-927f-166576284e86.png align="center")
    
    **STAGE-3 : SCAN THE SOURCE CODE: LOGIN INTO SONAR GO TO MY ACCOUNT &gt;&gt; SECURITY &gt;&gt; ENTER A TOKEN NAME AND GENERATE A TOKEN**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744950584931/f7031015-8b52-4a34-a1e2-76ae90ad849f.png align="center")
    
    NOW INTEGRATE THE SONAR TO JENKINS MANAGE JENKINS &gt;&gt; CONFIGURE SYSTEM &gt;&gt; SONAR SERVER
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744950746645/7905bf37-ff69-4005-8d94-5de261c305d6.png align="center")
    
    stage("CQA") { withSonarQubeEnv('mysonar') { def mavenHome = tool name: "maven3", type: "maven" def mavenCMD = "${mavenHome}/bin/mvn" sh "${mavenCMD} sonar:sonar" } }  
    *AFTER SUCCESSFULL BUILD CHECK FOR ANY BUGS IN THE SONARQUBE DASHBOARD&gt;PROJECTS*
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744951943312/69658a80-3441-470f-8728-b9dd3c4eec72.png align="center")
    
    **STAGE-4: Upload WAR File to Nexus Artifactory**
    
    Create a new repository in Nexus:  
    • **Name:** `bhavya-repo`  
    • **Format:** `maven2 (hosted)`  
    • **Version Policy:** `Releases`  
    • **Deployment Policy:** `Allow Redeploy`
    
    Install the **Nexus Artifact Uploader** plugin in Jenkins.
    
    Go to **Jenkins → Pipeline Syntax**, choose **Nexus Artifact Uploader** from the dropdown.
    
    Fill in the required fields:  
    • Nexus URL  
    • Repository name (`bhavya-repo`)  
    • Group ID, Artifact ID, Version(You can find in the pom.xml file)  
    • Path to WAR file
    
    Click **Generate Pipeline Script** and add it to your Jenkinsfile
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744952395185/95a0f441-2324-45b1-b586-6d86d32cce50.png align="center")
    
    ```plaintext
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
    
    *AFTER SUCESSFULL BIULD YOU CAN SEE THE WAR FILES UPLOADED IN NEXUS*
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744952963685/53f1b027-94ba-49d4-9979-19997e853f16.png align="center")
    
    **STAGE-5 : DEPLOY THE APPLICATION INTO TOMCAT WEB SERVER:**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744953168261/d760aae7-ba21-4f73-9635-abe933260941.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744953191203/2812dc8a-6261-4ca6-b8de-5dc116d559a9.png align="center")
    
    (Put your pem file privte key)
    
    ```bash
    #!/bin/bash
    
    # Move Tomcat folder to ec2-user home
    mv apache-tomcat-9.0.104 /home/ec2-user/
    
    # Change ownership to ec2-user
    chown ec2-user:ec2-user /home/ec2-user/apache-tomcat-9.0.104 -R
    
    # Switch to tomcat directory
    cd /home/ec2-user/apache-tomcat-9.0.104/bin
    
    # Stop Tomcat if running
    ./shutdown.sh
    
    # Start Tomcat
    ./startup.sh
    
    # Navigate to webapps folder
    cd ../webapps
    pwd
    ```
    
    ```plaintext
    stage("Deployment") {
        sshagent(['8bce0b0e-c5db-4aac-bf07-4831ca13a760']) {
            sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@52.23.240.218:/home/ec2-user/apache-tomcat-9.0.104/webapps/'
        }
    }
    ```
    
    *AFTER SUCCESSFULL BUILD YOU CAN SEE YOUR WAR FILED DEPLOYED INTO TOMCAT SERVER*
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744955838076/f551623a-3135-4318-bdd6-afb49b903235.png align="center")
    
    *CLICK ON YOUR WAR FILE TO VIEW YOUR APPLICATION*
    

<iframe src="https://drive.google.com/file/d/1VXveO2Sa9ZyhlAniLNATR3eiGrEabdZg/preview" width="640" height="480"></iframe>

### Final Pipeline Flow Summary

**From Code to Deployment:**

1. Developer pushes code to Git
    
2. Jenkins job triggers → SonarQube analysis
    
3. Maven builds code → Nexus stores WAR file
    
4. Jenkins deploys to Tomcat → Application is live
    

---

### ✅ What I Learned

* How each DevOps tool works in real-world projects.
    
* Jenkins integration with multiple tools for continuous integration and delivery.
    
* Managing AWS instances and implementing security best practices.
    
* Debugging and automating deployment steps to enhance workflow efficiency.
    

---

### 🔗 Resources & Links

* 📁 [https://github.com/PasupuletiBhavya](https://github.com/PasupuletiBhavya)
    
* 🖥️ **Tool Sites:**
    
    * [Jenkins](https://www.jenkins.io/)
        
    * [SonarQube](https://www.sonarqube.org/)
        
    * [Nexus](https://www.sonatype.com/)
        
    * [Apache Tomcat](https://tomcat.apache.org/)
        

---

This hands-on project provided me with a deeper understanding of DevOps pipelines and toolchains. For anyone just starting with DevOps, building a project like this is an invaluable way to learn and get practical experience. It also makes a great addition to your resume!If this article added value to your journey, your support would mean the world to me — only if it's within your means. Let's stay connected on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/). Thank you for reading, and let's keep growing together! 💕

💬 Feel free to connect or comment if you have questions or want to collaborate!