---
title: "End-to-End Docker Project with Jenkins CI/CD | Deploying a Node.js App with Security Scans"
seoTitle: "Build a Complete Docker CI/CD Pipeline with Jenkins, SonarQube, Trivy"
seoDescription: "A practical DevOps project for hands-on learning and portfolio building."
datePublished: Fri May 23 2025 03:28:36 GMT+0000 (Coordinated Universal Time)
cuid: cmb08r3e1000q09jshf0z6sm6
slug: end-to-end-docker-project-with-jenkins-cicd-deploying-a-nodejs-app-with-security-scans
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747971726993/06a792a9-6082-40a8-82de-2ad051da2a0e.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1747970694927/421d8d56-a911-413c-a80c-2c827bc5dfc7.png
tags: owasp, docker, sonarqube, git, npm, automation, jenkins, docker-compose, ci-cd, docker-images, devops-articles, trivy

---

# **Introduction**

This project demonstrates how to build a complete DevOps pipeline for a Node.js application using Jenkins, Docker, Trivy, SonarQube, and OWASP security checks. The pipeline automates every step from pulling the code from GitHub to deploying a secure Docker container.

You will learn how to:

* Automatically check out code and analyze it for bugs and code quality with SonarQube
    
* Install dependencies and scan them for security risks using OWASP best practices
    
* Build and scan Docker images for vulnerabilities with Trivy
    
* Push secure Docker images to Docker Hub and deploy them
    

This project helps you create an automated, secure, and reliable workflow to deliver high-quality applications faster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747891475212/62f916ec-4d7d-46da-ad95-4a953e78adc4.png align="center")

## Step-by-Step Process to Deploy the Application:

### Step 1: Launch EC2 Instance

Start by launching an EC2 instance on AWS. I used a **t2.large** instance and named it **DOCKER PROJECT**.

* OS: Amazon Linux 2
    
* Instance Type: t2.large
    
* Opened port 22 for SSH access
    

Once it's running, note the **public IP address** — you'll need it to connect via SSH.

### Step 2: Install Jenkins, Git, Docker & Trivy

To save time and streamline setup, I created a shell script [`jenkins.sh`](http://jenkins.sh) to install Git, Jenkins, Java, and Docker in one go.  

```bash
#STEP-1: INSTALLING GIT
yum install git -y

#STEP-2: GETTING THE REPO (jenkins.io --> download --> redhat)
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade -y
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-21-openjdk -y
sudo yum install jenkins -y
sudo systemctl daemon-reload

#STEP-3: DOWNLOAD JAVA17 AND JENKINS
yum install java-17-amazon-corretto -y
yum install jenkins -y

#STEP-4: RESTARTING JENKINS
systemctl start jenkins.service
systemctl status jenkins.service

#STEP-5: INSTALL DOCKER
yum install docker -y
systemctl start docker
```

Then run it:

```bash
sh jenkins.sh
```

This script automates the setup for Jenkins, Git, Java, and Docker on your EC2 instance.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747958702277/85a46ede-e068-48ea-a6b2-0420a522cbbe.png align="center")

You can see the Jenkins is active and running, access Jenkins using the IP address and port number 8000

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747958679300/de83738b-0cf3-4a85-8551-7e251a3bd8c7.png align="center")

Check if Git and Docker are installed properly.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747958653789/8f1c4732-43b6-4fc9-b6fa-c9278182083d.png align="center")

To let Jenkins run Docker commands, you need to set proper permissions on the Docker socket.

```bash
chmod 777 /var/run/docker.sock
```

⚠️ **Note:** This is fine for **testing**, but **not recommended for production**.  
A more secure approach is to add the Jenkins user to the Docker group

### Install Trivy:

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
```

At this point, the `trivy` command might not work if the path isn't set properly.

### Fix the PATH:

```bash
# Show hidden files
ll -a

# Open .bashrc
vim ~/.bashrc

# Add this line at the end
export PATH=$PATH:/usr/local/bin/

# Save and apply changes
source ~/.bashrc
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747959377871/5f31e168-e748-4a2e-8f4e-5e5afe4d5339.png align="center")

### Step 3 : Install Jenkins Plugins

* **SonarQube Scanner** – for code quality analysis
    
* **NodeJS** – to support Node-based applications
    
* **OWASP Dependency-Check** – to detect vulnerable dependencies
    
* **Docker Pipeline** – enables Docker support in Jenkins pipelines
    
* **AdoptOpenJDK (Temurin)** – for Java-based environments
    

### How to install them:

1. Go to **Jenkins Dashboard**
    
2. Click **Manage Jenkins → Manage Plugins**
    
3. In the **Available** tab, search for each plugin
    
4. Select them and click **Install without restart**
    

💡 Restart Jenkins after installation to ensure all plugins load properly.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747959933420/5be45b48-573d-4e0f-9482-b588f1dd85aa.png align="center")

### Step 4: Set Up SonarQube with Docker

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

This pulls the latest LTS community version of SonarQube and runs it in a container named `sonar`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747960237941/8bdd9d55-b4dc-448c-b7b0-afed2c1662f7.png align="center")

  
Access the SonarQube using the IP address with port number 9000. Username: `admin` Password: `admin`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747960433140/66d9f97a-58a8-46fc-a022-a768d50e16c6.png align="center")

*Create a dummy project to access the sonar dashboard*

### Step 4: Configure Jenkins Credentials and Tools

Now that SonarQube is up and running, let’s configure it with Jenkins by adding credentials, setting up the server, and linking the scanner.

Generate SonarQube Token:

1. Go to your **SonarQube dashboard**
    
2. Navigate to: `Administration → Security → Users → Tokens`
    
3. Click **Generate Token**, give it a name and copy the token
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961044577/f0fc00a8-7b36-438b-b5ba-457819308003.png align="center")

**Add Credentials in Jenkins:**

1. Go to **Jenkins Dashboard → Manage Jenkins → Credentials**
    
2. Choose **Global credentials** and click **Add Credentials**
    
3. Set:
    
    * **Kind:** Secret text
        
    * **Secret:** *Paste the SonarQube token*
        
    * **ID:** `sonar`
        
    * **Description:** `sonar password`
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961164723/06c94804-1fa4-453d-b5d1-53f9e323b85f.png align="center")

**Configure SonarQube Server in Jenkins:**

1. Go to **Manage Jenkins → Configure System**
    
2. Scroll to **SonarQube servers**
    
3. Click **Add SonarQube**
    
    * **Name:** `mysonar`
        
    * **Server URL:** `http://<your-sonarqube-ip>:9000`
        
    * **Server authentication token:** Select the `sonar password` you just added
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961313441/276d45ce-4d7a-4f17-97cc-00cf4168c126.png align="center")

Add Webhook in SonarQube:

1. Go to **SonarQube → Administration → Configuration → Webhooks**
    
2. Click **Create**
    
    * **Name:** `Jenkins`
        
    * **URL:** `http://<your-jenkins-ip>:8080/sonarqube-webhook/`
        

**Set up SonarQube scanner, NodeJS (node16), JDK (jdk17), and OWASP DP-Check**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961701785/86ec5e02-7cc8-4244-b8aa-71186b5a5151.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961724085/738a7fcd-a081-42ac-afdb-6c2921b32e6f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961743875/ab8cb752-0584-47c3-a9b0-c498154e7149.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747961766401/7fb32e4f-d52d-4477-be9a-ea68de21f978.png align="center")

### Step 5: Declarative Jenkins Pipeline

```bash
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
        stage ("Clean") {
            steps {
                cleanWs()
            }
        }

        stage ("Code") {
            steps {
                git "https://github.com/PasupuletiBhavya/Zomato-Project.git"
            }
        }

    }
}
```

BUILD THIS PIPELINE

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747962218993/539b3820-b1e8-4b2c-91c8-c01e3625090a.png align="center")

This confirms that the `Zomato-Project` has been cloned successfully into:

```bash
/var/lib/jenkins/workspace/MyDeployment/ 
```

```bash
stage("SCAN") {
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
```

SAVE AND BUILD

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747962512556/577f0a0c-50c8-49da-8d36-9b2fb3a41ac5.png align="center")

  
After successful build open SonarQube to check for bugs or vulnerabilities

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747962560005/6fc62305-5c59-482a-a755-97a0bb601b57.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747962672944/420b2282-c6f5-4a5d-91f0-a4daebe44c20.png align="center")

```bash
stage ("Quality Gates") {
    steps {
        script {
            waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
    }
}
```

This stage checks whether the code passed all the **SonarQube quality checks**. If it fails and `abortPipeline` is set to `true`, Jenkins will stop the build.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747962866515/c2d0b4e5-344c-443f-a239-137e2f252330.png align="center")

```bash
stage ("Install dependencies") {
    steps {
        sh 'npm install'
    }
}
```

This stage installs all required Node.js packages using `npm install`. This uses the `package.json` file to fetch and install all project dependencies before the build or run steps begin.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963054254/6a7cdb88-4567-4262-bb51-e62ad8914b99.png align="center")

After running `npm install`, the `node_modules` folder appears, confirming that dependencies were installed successfully.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963171947/d39e48d5-aa63-4728-a3f2-675485f494c3.png align="center")

This part of the pipeline covers security checks, Docker build, image push, scan, and app deployment.

**Final Pipeline Stages**

```bash
stage ("OWASP") {
    steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dp-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}

stage ("Trivy scan") {
    steps {
        sh "trivy fs . > trivyfs.txt"
    }
}

stage ("Build Dockerfile") {
    steps {
        sh 'docker build -t image1 .'
    }
}

stage("Docker Build & Push") {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-password') {
                sh "docker tag image1 bhavyap007/dockerproject:zoamto"
                sh "docker push bhavyap007/dockerproject:zoamto"
            }
        }
    }
}

stage ("Scan image") {
    steps {
        sh 'trivy image bhavyap007/dockerproject:zoamto'
    }
}

stage ("Deploy") {
    steps {
        sh 'docker run -d --name zomato -p 3000:3000 bhavyap007/dockerproject:zoamto'
    }
}
```

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963818283/20f878b7-90c2-444d-8afe-313658a19ed2.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963844138/43fc935b-505a-437b-b85f-7ccc659999a3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963874229/a6ca90db-fa47-4dba-8dff-4ed2c67667c7.png align="center")

All stages executed successfully, completing the CI/CD pipeline from code to container to deployment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747963959847/8e67fd92-0606-4108-82f8-9a71181e8547.png align="center")

After building and tagging the Docker image, it was successfully pushed to Docker Hub under the repository:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1747964113602/aca6bb29-ca6f-4144-bc13-c8959cf7ef7e.png align="center")

The image is now public and can be pulled and deployed from anywhere using:

> ```bash
> docker pull bhavyap007/dockerproject:zomato
> ```

Access your application with your IP address and port number 3000

<iframe src="https://drive.google.com/file/d/1SfiD_lUYvSbhqlNPDhcmtxNvVZVKURly/preview" width="640" height="360">
</iframe>

## Final Pipeline Flow Summary

1. **Launch EC2 instance** and open required ports.
    
2. **Install Jenkins, Git, Docker, Trivy, and OWASP tools.**
    
3. **Configure Jenkins plugins** and add SonarQube token.
    
4. **Run SonarQube using Docker** and link it to Jenkins.
    
5. **Create Jenkins pipeline** with these stages:
    
    * Clean workspace
        
    * Clone code from GitHub
        
    * Run SonarQube scan
        
    * Wait for Quality Gate result
        
    * Install Node.js dependencies
        
    * Run OWASP and Trivy scans
        
    * Build and push Docker image
        
    * Scan Docker image
        
    * Deploy container
        
6. **App runs on port 3000** — deployment successful!
    

## **✅**What I Learned

* How to set up a full CI/CD pipeline using **Jenkins** and **Docker**.
    
* Installing and configuring tools like **SonarQube**, **Trivy**, and **OWASP Dependency-Check**.
    
* Writing a **Declarative Jenkins Pipeline** from scratch.
    
* Scanning code and Docker images for vulnerabilities.
    
* Automating Docker image builds, pushes to Docker Hub, and deployments.
    
* How all DevOps tools connect together to ensure smooth, secure delivery
    

## 🔗 Resources & Links

* 📁 [**https://github.com/PasupuletiBhavya**](https://github.com/PasupuletiBhavya)
    
* 🖥️ **Tool Sites:**
    
    [Jenkins](https://www.jenkins.io/)  
    [Docker](https://www.docker.com/)  
    [SonarQube](https://www.sonarsource.com/products/sonarqube/)  
    [Trivy](https://aquasecurity.github.io/trivy/)  
    [OWASP Dependency-Check](https://jeremylong.github.io/DependencyCheck/)  
    [Git](https://git-scm.com/)  
    [Node.js](https://nodejs.org/)
    

---

This hands-on Docker CI/CD project gave me a deeper understanding of how real-world DevOps pipelines work — from source code management to automated testing, security scanning, containerization, and deployment. It brought together tools like Jenkins, Docker, SonarQube, Trivy, and OWASP Dependency-Check in a complete flow.

For anyone starting out in DevOps, building a pipeline like this is one of the best ways to gain **practical, resume-worthy experience**.

If this article helped you in any way, your support would mean a lot to me 💕 — only if it's within your means.

Let’s stay connected on [LinkedIn](https://www.linkedin.com/in/bhavya-pasupuleti/) and grow together!

💬 Feel free to **comment or connect** if you have questions, feedback, or want to collaborate on similar projects.