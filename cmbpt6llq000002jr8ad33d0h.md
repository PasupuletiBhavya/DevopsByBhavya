---
title: "CI/CD Pipeline with Docker: DevSecOps for Real-World App Deployment"
seoTitle: "Docker-Focused Deployment Only"
seoDescription: "Docker-Focused Deployment Only"
datePublished: Tue Jun 10 2025 00:54:47 GMT+0000 (Coordinated Universal Time)
cuid: cmbpt6llq000002jr8ad33d0h
slug: cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1753905658448/cd459bd3-1dd7-4135-84c7-258e9c0d54ac.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1749774705234/2480f345-c54f-4c69-a137-bd76540811b2.png
tags: docker, github, deployment, sonarqube, kubernetes, slack, automation, devops, terraform, jenkins, devsecops, ci-cd, docker-images, devops-articles, trivy

---

A practical DevSecOps pipeline built entirely with Docker and Jenkins — from code to cloud.

<mark>Missed the full DevSecOps journey?</mark>  
👉 [Start here with the full 6-phase blog](https://devops-by-bhavya.hashnode.dev/campus-drive-management-a-complete-devsecops-journey-6-phases-explained)

## [Understanding the 3-Tier Application](https://chatgpt.com/c/684384bf-6818-8010-93ad-a05e1740dcde)

In this project, we’re working with a **three-tier web application** architecture, commonly used in modern full-stack development. The application is called **Yelp Camp** — a dynamic campground listing platform.

1. **Frontend (Client-side UI):**  
    The visual interface where users interact with the app — creating, viewing, and reviewing campgrounds.
    
2. **Backend (Server-side Logic):**  
    Handles user requests, manages authentication, routes data, and applies business logic.
    
3. **Database (Data Storage):**  
    Stores campground details, user information, images, and reviews.
    

---

## What is Yelp Camp?

**Yelp Camp** is a full-stack web application that enables users to:

* Register and log in
    
* Create and review campgrounds
    
* Upload images using **Cloudinary**
    
* Display locations dynamically via **Mapbox**
    

### Key Features:

* User registration (no email verification required)
    
* Form validation for unique email
    
* MongoDB database for storing campground data
    
* Map-based campground UI
    
* Review system (users can only delete their own reviews)
    
* Dynamic image uploads via **Cloudinary**
    

## Database & Deployment Strategy

### Option 1: MongoDB as a Docker Container

**(Not used in our final setup, but worth understanding)**

* **Requires:**
    
    * Deployment YAML to create the MongoDB pod
        
    * Service to expose the pod
        
    * Persistent Volume (PV) and Persistent Volume Claim (PVC) for data retention
        
* **Drawbacks:**
    
    * Manual setup
        
    * Backup risk on pod crash
        
    * Storage cost and maintenance
        

### ✅ Option 2: MongoDB Atlas (Cloud Database) — *Our Choice*

* **Benefits:**
    
    * Fully managed cloud MongoDB
        
    * No need for K8s pods, services, or volumes
        
    * UI-based dashboard for management
        
    * Reliable, scalable, and simple setup
        
* **Downside:**
    
    * Paid plan (Free tier available for small projects)
        

---

## Required Environment Variables

To get this app running, we need to configure the following secrets in `.env` or Jenkins credentials:

### Cloudinary:

```bash
CLOUDINARY_CLOUD_NAME=your_cloud_name  
CLOUDINARY_KEY=your_api_key  
CLOUDINARY_SECRET=your_api_secret
```

### Mapbox:

```bash
MAPBOX_TOKEN=your_mapbox_token
```

### MongoDB Atlas:

```bash
DB_URL=your_mongodb_connection_string
```

## Setting Up Cloudinary for Image Uploads

To enable image uploads in our Yelp Camp application, we’ll integrate **Cloudinary** — a cloud-based image and video management service.

1. Go to [https://cloudinary.com](https://cloudinary.com) and **log in** (or sign up).
    
2. From the dashboard, click **View All API Keys.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749503792941/2a14cefc-5262-42a6-b7cd-4d225caffd07.png align="center")
    
3. You’ll find the following credentials:
    
    * **Cloud Name**
        
    * **API Key**
        
    * **API Secret**
        
    * **API Environment Variable**
        
4. **Copy these credentials and store them securely** — we’ll use them as environment variables in the application.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749503898785/671695e2-3a96-4607-a722-bae6d16bb29b.png align="center")

### Setting Up Mapbox for Location Mapping

To display campground locations on an interactive map, we use **Mapbox** — a powerful mapping platform for developers.

1. Go to https://account.mapbox.com and **log in** (or create an account).
    
2. Under your account dashboard, you’ll find a **“Default Public Token.”**
    
3. **Copy this token** — it’s all you need for this application.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504005317/f42d5960-abb1-4e35-8a03-427cd592457d.png align="center")

## Setting Up MongoDB Atlas (Cloud Database)

To store all campground data (user info, reviews, locations), we use **MongoDB Atlas** — a fully managed cloud database service.

1. Go to [https://www.mongodb.com/cloud/atlas and **log in** or create an account.](https://www.mongodb.com/cloud/atlas)
    
2. **"Create a Cluster"** to start a free-t[ier or shared cluster deployment.](https://www.mongodb.com/cloud/atlas)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504269992/378e2258-c5a1-4ada-b153-b011109f0666.png align="center")

3. After your cluster is created, click **"Create Database User"**:
    

* Choose a username & password
    
* **Save them securely** (we’ll use these for your app connection string)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504330341/09e730ca-de8f-45e3-aa71-71d20109296b.png align="center")

4. ### Choose a Connection Method:
    
    1. Click on **“Connect” → “Drivers”**
        
    2. Select **Node.js** and copy the **MongoDB connection URL** provided
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504464017/4a807396-e64f-4f28-a534-d8217166ef20.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504476593/528db870-6c30-4bc8-8933-5c72e77756e6.png align="center")
    
    ### Enable External Access:
    
    By default, MongoDB Atlas restricts access to [localhost](http://localhost). To make it accessible:
    
    1. Go to **Network Access** in your Atlas dashboard
        
    2. Click **“Add IP Address”**
        
    3. Choose **“Allow access from anywhere” (0.0.0.0/0)**
        
    
    ✅ This step ensures your app, running from any server (like EC2), can talk to the DB.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749504979035/d6387e74-9dab-423f-add7-e7a4589ef0b1.png align="center")

## Step-1: EC2 Instance Setup (AWS)

* Create a `t2.large` EC2 instance with **28 GB** storage and **Linux Kernel 5.10**
    
* Attach a **key pair** for SSH access and configured a **security group** (ports 22, 80, 8080)
    
* Launch instance to install Docker, Jenkins, and run our application.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749505025094/ab5d7b93-ce25-43b1-87ec-a52f766f74c3.png align="center")

If you're new to Jenkins or Docker setup, check out my detailed guides:

👉 [End-to-End Docker Project with Jenkins CI/CD (Node.js + Trivy)](https://devops-by-bhavya.hashnode.dev/end-to-end-docker-project-with-jenkins-cicd-deploying-a-nodejs-app-with-security-scans)  
👉 [CI/CD Pipeline with Jenkins on AWS (EC2 Setup Guide)](https://devops-by-bhavya.hashnode.dev/end-to-end-cicd-pipeline-with-jenkins-on-aws)

## Step 2: Install Jenkins, Git, Docker, Terraform, Trivy and Access the Jenkins Dashboard

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749505443072/122211ea-d932-4325-97c3-c5ab7702dd8f.png align="center")

## Step-3: Dockerize the Application

### 1\. Clone the Project Repository

```bash
git clone https://github.com/PasupuletiBhavya/devsecops-project.git
cd devsecops-project/
git checkout master
```

### 2\. Build the Docker Image

```bash
docker build -t image1 .
docker images
```

### 3\. Run the Docker Container

```bash
docker run -itd --name cont1 -p 1111:3000 image1
```

But when we checke the logs:

```bash
docker logs cont1
```

We this error:  
Error: Cannot create a client without an access token

### ⚠️ What Happened?

Even though the container started, the app **crashed inside** because it couldn't find the required credentials for:

* **Mapbox** (for displaying maps)
    
* **Cloudinary** (for uploading images)
    
* **MongoDB Atlas** (for storing application data)
    

These are **not hardcoded** in the app. They're expected to be passed as **environment variables**

### ✅ How to Fix It

We fix this by passing all the necessary env variables when starting the container:

```bash
vim .env
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_KEY=
CLOUDINARY_SECRET=
MAPBOX_TOKEN=
DB_URL=""
SECRET=my key
```

Now the container runs properly and the app can talk to all 3rd-party services.

```bash
docker rm cont1
docker build -t image2 .
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506575645/985edfd6-d91e-4dbe-a9bf-34d562eccd30.png align="center")

Access the application with port number

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506626430/94823b6b-3f96-4ccf-ad6e-2656caee04d6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506651412/1c0f6ab9-6526-4b73-a97e-ae3517c91a94.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506712448/80634d66-c07d-4e17-84c9-3f0f039dc809.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506669469/67f0ed43-0ba4-4ed0-8be8-d05d6f6b97a9.png align="center")

You can find all these data updated in our MongoDB

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506825883/a12e2ee6-efb6-4bd1-b428-9deb8814abd9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749506855360/bc6e54e9-8bb6-48eb-b177-bcc8db44c9a9.png align="center")

Things to consider:  
When building our Docker image for the application, we used the **Node.js Alpine image** instead of the default Node image.

```bash
# Use Node 18 as parent image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy remaining app files
COPY . .

# Expose port
EXPOSE 3000

# Start the app
CMD npm start
```

### Why Use Alpine?

✅ **Smaller size**: Alpine-based image is only **~196MB**, whereas a default Node image may be over **1 GB**

✅ **Faster deployments**: Smaller image = faster builds & pushes

✅ **Security**: Fewer libraries = smaller attack surface

✅ **Great with Trivy**: No vulnerabilities found during scan

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749507435705/f65e64f7-2dbd-441d-b49f-4068d8a42f8a.png align="center")

### What Happens Without Alpine?

Image size becomes 1.1GB+

Base image pulls in unnecessary dependencies

Trivy detects more vulnerabilities

Slower builds, more risk

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749507559514/0a929542-4f83-4f56-adef-22fd6628ed5a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749507575922/8600d4c4-9dcd-4a73-8bb5-ddd83d9e2ad9.png align="center")

Always use lightweight base images like `node:18-alpine` for faster, smaller, and more secure containers.  
Now let’s move on **Automation with Jenkins CI/CD**.

## **Step-4 :Automate Dev Server Provisioning Using Terraform**

To avoid manually launching EC2 instances from the AWS console, we use **Terraform** to create our **Dev server** automatically as code. This brings repeatability, version control, and automation into our infrastructure.

**Create Infra Directory**

```bash
mkdir infra
cd infra
```

**create** [`provider.tf`](http://provider.tf)

This file tells Terraform which cloud provider and region we want to use.

```bash
provider "aws" {
  region = "us-east-1"
}
```

**Create** [`resource.tf`](http://resource.tf)

```bash
resource "aws_instance" "devserver" {
  ami           = "ami-0e9bbd70d26d7cf4f"  # Amazon Linux 2 AMI
  instance_type = "t2.medium"
  key_name      = "master-slave"
  availability_zone = "us-east-1a"

  root_block_device {
    volume_size = 20
  }

  tags = {
    Name        = "Camp-Server"
    Environment = "Dev"
    Client      = "bhavya"
  }
}
```

### <mark>Give Terraform Permission to Create AWS Resources</mark>

Terraform itself can’t create anything — it's just a CLI tool. To let it create AWS resources like EC2 instances, we need to **authenticate it with our AWS account**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749508593555/ba114d5a-eee8-445f-bfa1-5027ed5a6b6f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749508606888/2e9909b9-f39d-49ed-ae7b-4aaeeac7bc2f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749508628059/c6d6a943-d001-465a-a52d-8f70c46c32b8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749508649750/ca87ec67-a21b-4724-85c9-68ac7a5a1459.png align="center")

```bash
terraform init
terraform plan
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749509114785/5014ea97-231a-4439-a158-33e6b3737ce8.png align="center")

we create our Dev server by applying the `.tf` files:

```bash
terraform apply --auto-approve
```

✅ This command automatically provisions the EC2 instance without asking for confirmation every time

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749509226646/f98bf7b1-2f62-45fd-b780-1eddd834ff35.png align="center")

*update the security group*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749509244433/947bfdaa-a2ff-4d82-b02e-4a2591e07ddd.png align="center")

```bash
#Install java on dev server
yum install java-17-amazon-corretto -y
```

## **Step-5 : Master-Slave setup**

Until now, we’ve been walking through the steps manually — setting up infrastructure, containerizing applications, and deploying them to the cloud. But in real-world scenarios, **DevOps Engineers automate all of this** using CI/CD pipelines.

As a DevOps Engineer, I use **Terraform** to provision infrastructure — for example, creating the **Dev server** (EC2 instance) automatically.

But now, we need to execute our Jenkins pipeline **on that server**. For that, we need a **Master-Slave setup**:

* **Jenkins Master:** Where the pipeline is written and controlled
    
* **Jenkins Slave (Agent):** Where the actual pipeline runs — in our case, the EC2 Dev server.
    

Go to Manage Jenkins →Nodes → Add new Node

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510281817/4595c900-baeb-4ccd-b513-51ab3908ceeb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510315399/21aaaac6-6cd1-4922-bf7f-894dd51f8f3f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510333418/1181d7f1-9fe6-4751-95d6-e2b1df221904.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510348154/a4ca33dd-5eec-4a50-9f8b-d49883d702de.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510378803/f8178d13-9e7a-4743-b4d3-1a1faf925cd1.png align="center")

**<mark>Install all necessary plugins ,Configure Jenkins Credentials and Tools</mark>**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510547913/ce670a36-562b-4844-b9c8-016e8dac51c2.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510563051/e12d3ac0-df35-4b27-8fb6-b41094f99c7f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510574485/08318375-3ca9-474c-b8a8-7d27427d4691.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510586496/7fdc260c-869c-49cf-b6ff-8d20630c0a5b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749510619958/0a6b731c-c82e-490d-a1c5-5e6747005a98.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749511036980/25a03ce1-bd1c-4d76-93ca-2cf3f51f6460.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749511052824/853a88b0-f1cf-41be-8526-a18a0a39515f.png align="center")

**Lets write our pipeline**  
Create a new JOB → Pipeline → start writing our pipeline

```bash
pipeline {
    agent {
        node {
            label 'dev'
        }
    }
    tools {
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {
        stage('Code Checkout') {
            steps {
                git "https://github.com/PasupuletiBhavya/devsecops-project.git"
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=camp \
                    -Dsonar.projectKey=camp
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-password'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t appimage .'
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh 'trivy image appimage'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag appimage bhavyap007/newproject:dev-v1'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker push bhavyap007/newproject:dev-v1'
                    }
                }
            }
        }

        stage('Deploy to Dev Server') {
            steps {
                sh 'docker run -itd --name dev-container -p 1111:3000 bhavyap007/newproject:dev-v1'
            }
        }
    }
}
```

BUILD THIS PIPELINE

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749511334593/9403c5e5-8aa6-4752-879a-a2e8d567b7ba.png align="center")

After successful build open SonarQube to check for bugs or vulnerabilities

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749511367240/4fd5212d-e38f-4e68-9c41-d7b7642184b0.png align="center")

Access your application with your IP address and port number

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749511438529/c2afacb9-0d67-4cf0-8c85-80214254acaa.png align="center")

## **Step-6 :Automate Testing Server Provisioning Using Terraform**

To isolate environments like Dev, Test, and Prod using the same Terraform code, we use **Terraform Workspaces**. This allows us to manage multiple infrastructure environments from a single codebase.

1. **Modify** [`resource.tf`](http://resource.tf) to reflect the new environment:
    

```bash
  resource "aws_instance" "devserver" {
  ami           = "ami-0e9bbd70d26d7cf4f"
  instance_type = "t2.medium"
  availability_zone = "us-east-1a"
  key_name      = "master-slave"

  tags = {
    Name        = "test-Camp-Server"
    Environment = "test"
    Client      = "bhavya"
  }

  root_block_device {
    volume_size = 20
  }
}
```

2. **Create and switch to a new workspace**:
    

```bash
terraform workspace new test
```

3. **Apply the infrastructure**:
    

```bash
terraform apply --auto-approve
```

Server is created

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749512254882/0cb2093e-f563-4f88-8f8e-f39d61f0ef13.png align="center")

Setup Master-Slave as discussed above and create a node for testing

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749512299545/e80490b4-bc6d-4207-b1fe-ab2aedcc8aad.png align="center")

Create New Job for testing

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749512348753/e13a1726-8fc5-4646-a4e5-33115a10f05c.png align="center")

Install Jenkins, Git, Docker, Terraform, Trivy in your testing server as well

```bash
groovyCopyEditpipeline {
    agent {
        node {
            label 'test'
        }
    }
    tools {
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {
        stage('Code Checkout') {
            steps {
                git "https://github.com/PasupuletiBhavya/devsecops-project.git"
            }
        }

        stage('Code Quality Analysis') {
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
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-password'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t appimage .'
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh 'trivy image appimage'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag appimage bhavyap007/newproject:test-v1'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker push bhavyap007/newproject:test-v1'
                    }
                }
            }
        }

        stage('Deploy to Dev Server') {
            steps {
                sh 'docker run -itd --name test-container -p 2222:3000 bhavyap007/newproject:test-v1'
            }
        }
    }
}
```

BUILD PIPELINE

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749512640298/7ae34623-a303-428c-8fa9-7845b7627731.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513162471/cfd52b39-ed39-46a8-9113-914b1f072aab.png align="center")

## Step-7: Slack Notification in Jenkins Pipeline

To notify your team about the pipeline status, we use the **Slack plugin** in Jenkins.  
Install the **Slack Notification Plugin and configure**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513375959/b4d6dcb2-4e20-45ca-a7b1-9310b9987973.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513508456/4f26fdfd-b7f0-4765-ab7f-1f5771dbed12.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513542458/7db3bceb-6a7d-4dea-9829-8b7a13183ddf.png align="center")

Login into your slack account and integrate with your Jenkins

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513429615/61147860-5c07-4f03-b9c9-cc709856b3ff.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513447978/5d5a6175-b0f8-46ed-a4a1-7172fecc30aa.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513536577/dc2acb66-391c-4560-88b5-6ebb0e41623b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513465167/00a414c5-0ac3-434b-9cc4-d5ff1f606604.png align="center")

Now add slack in your pipeline under post build actions and build again

```bash
groovyCopyEditpost {
    always {
        echo 'Slack Notifications'
        slackSend(
            channel: 'my-channel',
            message: "*${currentBuild.currentResult}:* Job `${env.JOB_NAME}` \nBuild #${env.BUILD_NUMBER} \n🔗 More info: ${env.BUILD_URL}"
        )
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513697493/df02ca42-faca-4cf5-99cb-1046ee1acd6d.png align="center")

CHECK YOUR SLACK FOR UPDATES

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749513729501/6b9d248e-fe99-455d-9b74-60987eabda81.png align="center")

**❌ Drawbacks of Docker-only:**

* Delays during container restart (10s–1min) can hurt critical apps
    
* No easy rollback to older versions
    
* No built-in auto-scaling
    

**✅ Kubernetes Benefits:**

* **Auto-scaling** based on traffic
    
* **Rolling updates & Rollbacks**
    
* **Self-healing** containers
    
* **Cluster-based deployment** using master and worker nodes
    
* **Namespace isolation** for multi-app environments
    

⚠️ Docker has its limits in production. That’s why we switch to Kubernetes.  
<mark>Ready to roll into </mark> **<mark>Staging + Production?</mark>**  
👉 [Read my Kubernetes Deployment Blog here](https://devops-by-bhavya.hashnode.dev/scaling-devsecops-with-kubernetes-zero-downtime-auto-healing-and-more)

### **Final Pipeline Flow Summary (Dev + Testing)**

✅ Code pulled from GitHub

*Fetched full application codebase to start the build.*

✅ SonarQube Code Quality Analysis

*Performed static code analysis for bugs and vulnerabilities.*

✅ Docker Image Build (Alpine)

*Created a lightweight and optimized image using node:18-alpine.*

✅ Security Scan with Trivy

*Detected and ensured no critical vulnerabilities before shipping.*

✅ Tag & Push to Docker Hub

*Versioned image (dev-v1) was pushed to a Docker Hub repository.*

✅ Dev Deployment to EC2 Server

*Application container deployed to a dedicated Dev EC2 instance.*

✅ Slack Notifications

*CI/CD pipeline status updates sent directly to Slack channel.*

✅ Testing Server Setup (Terraform Workspace)

*Created separate Testing EC2 environment using Terraform.*

✅ Testing Pipeline Deployment

*Same image tested in a dedicated Testing instance for UAT, Regression, and Functional testing.*

### What I Learned

* **Importance of Lightweight Images:**  
    Using `node:18-alpine` significantly reduced image size and improved Trivy scan results.
    
* **End-to-End DevSecOps Flow with Docker:**  
    Built a secure, automated CI/CD pipeline from scratch using Jenkins.
    
* **Security is Not Optional:**  
    Trivy helped me catch vulnerabilities early before pushing to Docker Hub.
    
* **Infrastructure as Code (IaC):**  
    Learned how to provision EC2 dev servers using Terraform.
    
* **Team Collaboration with Slack:**  
    Seamless communication by integrating Jenkins with Slack.
    

---

For anyone starting out in DevOps, building a pipeline like this is one of the best ways to gain **practical, resume-worthy experience**.

If this article helped you in any way, your support would mean a lot to me 💕 — only if it's within your means.

Let’s stay connected on [**LinkedIn**](https://www.linkedin.com/in/bhavya-pasupuleti/) and grow together!

💬 Feel free to **comment or connect** if you have questions, feedback, or want to collaborate on similar projects.