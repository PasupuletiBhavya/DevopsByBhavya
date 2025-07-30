---
title: "Scaling DevSecOps with Kubernetes: Zero Downtime, Auto Healing, and More"
seoTitle: "Scaling DevSecOps with Kubernetes | Bhavya Pasupuleti"
seoDescription: "Hands-on guide to building a secure CI/CD pipeline using Jenkins, Docker, AWS, Kubernetes & Trivy — with zero downtime and auto-healing deployment."
datePublished: Thu Jun 12 2025 22:53:04 GMT+0000 (Coordinated Universal Time)
cuid: cmbtz5n9b000002gqbnn65fp2
slug: scaling-devsecops-with-kubernetes-zero-downtime-auto-healing-and-more
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1749768493469/3b3d5e0a-f049-47f3-ad5f-1d2d138e90d7.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1749774587123/5551dcf2-ff33-4218-bcc4-1a935f079a59.png
tags: docker, github, sonarqube, kubernetes, devops, terraform, jenkins, helm, devsecops, ci-cd, argocd, autoscaling, trivy

---

Take your CI/CD pipeline to the next level with Kubernetes-powered production deployments, built for scale, resilience, and real-world traffic.

### <mark>Missed the full DevSecOps journey?</mark>

👉 [**Start here**](https://devops-by-bhavya.hashnode.dev/campus-drive-management-a-complete-devsecops-journey-6-phases-explained) with the full 6-phase blog  
👉 **Or** [**check the Docker-only version first**](https://devops-by-bhavya.hashnode.dev/cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment)

# **Introduction:**

In this third phase of our DevSecOps journey, we take things to the next level with **Kubernetes**. While Docker helped us containerize and deploy our app, real-world production demands more:

* ⏱️ **Zero Downtime** during updates
    
* 🔁 **Rollbacks** when things go wrong
    
* 📈 **Auto Scaling** to handle load spikes
    
* ❤️ **Self-Healing** containers that recover automatically
    

Kubernetes provides all of that — and more.

In this blog, we’ll move from **Docker-only deployment** to a **robust Kubernetes setup**, covering:

* ✅ Kubernetes Basics (Clusters, Pods, Services)
    
* ✅ Deployment Strategies
    
* ✅ EKS Setup (AWS Managed K8s)
    
* ✅ Writing YAMLs (Deployment, Services, ConfigMaps)
    
* ✅ Final Production Push
    

## Step 1: Launch an EC2 Instance for Kubernetes Setup

We’ll start by launching a new EC2 instance where we’ll install and configure our Kubernetes environment.

* **AMI**: Amazon Linux 2 (Kernel 5.10)
    
* **Instance Type**: `t2.large`
    
* **Storage**: 25 GB EBS Volume
    
* **Key Pair**: Use your existing key pair (or create a new one)
    
* **IAM Role**: Attach the same IAM role used in your Docker-based setup  
    *(or create a new IAM role with EC2, S3, EKS full access if needed)*
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749678944032/2cc08ac3-8548-446c-9918-236284dbb012.png align="center")

> 💡 *Tip: Ensure port 22 (SSH) is open in your security group for accessing the instance.*

## **Step 2: Install the Complete DevSecOps Tech Stack**

### Install Git

```bash
yum install git -y
```

### Install Jenkins

```bash
# Add Jenkins repo
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Java 17 & Jenkins
yum install java-17-amazon-corretto -y
yum install jenkins -y

# Start and enable Jenkins
systemctl start jenkins
systemctl enable jenkins
systemctl status jenkins
```

### Install Docker

```bash
yum install docker -y
systemctl start docker
systemctl enable docker
systemctl status docker
chmod 777 /var/run/docker.sock
```

### Install Terraform or [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

```bash
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y
```

### Install SonarQube (Using Docker)

```bash
docker run -itd --name sonar -p 9000:9000 sonarqube:lts-community
```

### Install Trivy (Image Vulnerability Scanner)

```bash
# Update bashrc to include /usr/local/bin
vim ~/.bashrc
export PATH=$PATH:/usr/local/bin/
source ~/.bashrc

# Download and install Trivy
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
```

### Verify Installations

```bash
git --version
jenkins --version
docker version
terraform version
trivy --version
docker ps
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749679762226/23504828-6d84-47b6-9461-8b953e59441a.png align="center")

### Install AWS CLI (v2) or follow [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Install `kubectl` (Kubernetes CLI) or follow [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```bash
curl -LO "https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### Install `eksctl` (for creating EKS clusters) or follow [For Unix To download the latest release,](https://eksctl.io/installation/)

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" -o eksctl.tar.gz
tar -zxvf eksctl.tar.gz
sudo mv eksctl /usr/local/bin/
eksctl version
```

* `aws` – CLI for accessing your AWS account
    
* `kubectl` – To interact with your Kubernetes clusters
    
* `eksctl` – Simplifies EKS cluster creation and management
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749676614444/3e503560-dae3-4fed-b935-e5202377e45e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749676673203/9b0a66e1-3c18-4f37-8f01-46e565cb4276.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749676717800/6a13c4b0-d440-4c92-9420-589e4267e0f8.png align="center")

### Navigate to the Terraform Directory

```bash
git clone https://github.com/PasupuletiBhavya/devsecops-project.git
git checkout master
cd k8s-project/eks-terraform
```

Inside this folder, we have `.tf` files to define:

* The AWS region & provider
    
* The VPC, subnets, and EKS cluster
    
* IAM roles and node groups
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749677578406/cd8ab1f5-5118-4c90-8535-50f8afb8c41d.png align="center")

### **How Terraform Manages Infrastructure Internally?**

When we use Terraform to create resources (like AMI, instance type, EBS, IPs), it stores all the details in a **state file (**`terraform.tfstate`).  
What Terraform Stores:

✅ AMI IDs✅ Instance Types✅ EBS Volume Details✅ Key Pair names✅ Public & Private IPs✅ DNS Info✅ Subnet and VPC IDs✅ IAM roles✅ Security Groups

### **Why is the State File Important?**

The `.tfstate` file acts like a **source of truth** for Terraform.  
It keeps track of what has already been created and prevents duplication or drift.

Without it:

* Terraform won’t know what already exists
    
* You risk duplicating resources or breaking infrastructure
    

### **Why Store it in S3?**

Instead of keeping it locally, we **store the state file in an S3 bucket** so:

* It’s safe and backed up
    
* Team members can share it
    
* It avoids duplication or conflicts
    

## **Step-3: Update your** [`backend.tf`](http://backend.tf)**,** [`provider.tf`](http://provider.tf)**, and** [`main.tf`](http://main.tf) **files as shown below to configure Terraform with S3 backend and AWS provider before creating the EKS cluster**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749678461686/27047d0f-c795-4098-8836-1491a1bd261b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749678616433/7c259c87-33b6-40b1-a2d0-ffff77e83e15.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749678637954/f60395e7-9b31-4062-bf4c-373b129ac57d.png align="center")

After updating your [`backend.tf`](http://backend.tf), [`provider.tf`](http://provider.tf), and [`main.tf`](http://main.tf) files:

```bash
#Initializes Terraform, downloads providers, and configures the S3 backend.
terraform init
# Shows a preview of what resources will be created, updated, or destroyed.
terraform plan
#Provisions the entire EKS infrastructure without manual confirmation
terraform apply --auto-approve
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749679098268/cdc04d90-a812-4ddf-ac04-613af68a7ae8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749678985653/f6819fbb-5530-4f8c-bddc-21b46cf284c3.png align="center")

## Step 4: Connect to EKS Cluster

After creating the EKS cluster with Terraform, follow these steps:

```bash
#Check if cluster exists:
eksctl get cluster --region us-east-1
```

You should see your cluster listed. If `EKSCTL CREATED` is `False`, that means it was created via Terraform (as expected!).

```bash
#Connect kubectl to EKS:
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
```

> This sets up your kubeconfig so `kubectl` can talk to the cluster.

```bash
#Verify nodes
kubectl get nodes
```

✅ Output should show your worker node(s) in **Ready** state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749679560196/0501f088-6d78-4d45-a9fa-7b0551b69b37.png align="center")

***<mark>Our Cluster and server is Created!!</mark>***

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749679668883/4044eee7-a229-4a97-9bde-9ba769730b6a.png align="center")

### So, Why Did You See a Server Created When You Made a Cluster?

That was a **worker node EC2** that EKS created for you using a Node Group. That EC2:

* Belongs to your cluster
    
* Hosts your app containers (inside Kubernetes pods)
    
* Is **not your Jenkins server**
    
* Is **not the control plane**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749679682630/dd2ce206-cc47-40e3-a9a1-24038f1548fd.png align="center")

### *💡****Understanding Our Setup: Ops Server vs Pre-Prod Cluster:***

**Ops Server (Jenkins EC2 Instance):**  
This acts as our **control center** — where Jenkins is installed and all our **DevSecOps pipelines** are written and triggered.  
It automates everything: code checkout, scans, image builds, testing, deployments, and even Slack notifications.

**Pre-Prod Cluster (EKS\_CLOUD):**  
This is our **Kubernetes cluster** provisioned via Terraform & `eksctl`.  
It serves as the final **staging and production environment**.  
👉 After testing in Dev & QA, the same Docker image is deployed ,first in staging, then in production.

* **Dev** = Developer testing
    
* **UAT** = Client-side testing (QA)
    
* **Staging** = Pre-prod environment
    
* **Prod** = Final deployment
    

### Environment-wise CI/CD Pipeline Structure

To maintain control, flexibility, and rollback options, we create dedicated pipelines for each environment:

#### 🔹 UAT Environment (Internal Testing by QA or Clients)

* **1 Build Pipeline**: Builds and tags Docker image (UAT-specific version)
    
* **1 Deploy Pipeline**: Deploys the UAT image to the UAT namespace in EKS
    

#### 🔹 Staging Environment (Final Testing Before Production)

* **1 Build Pipeline**: Builds and pushes the staging Docker image
    
* **1 Deploy Pipeline**: Deploys the same image to the staging namespace
    

#### 🔹 Production Environment (Live Users)

* **1 Deploy Pipeline** only:  
    No rebuild is done — **the same image from staging is reused** to ensure stability and traceability
    

Now access your Jenkins dashboard with eks-server IP address and port number 8080:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749680866995/e85abbfe-d2a9-4ead-82fc-6d3d53b22d85.png align="center")

**<mark>Install all necessary plugins ,Configure Jenkins Credentials and Tools</mark>**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749680928248/1ec47327-9c83-4cf8-a849-e7cc1b51e5f9.png align="center")

You can refer to 👉 [my previous blogs for configuration setup](https://devops-by-bhavya.hashnode.dev/cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment)

## Step-5 : **Lets write our pipeline**

Create a new JOB(UAT\_Build\_Deployement)→ Pipeline → start writing our pipeline  
This Jenkins pipeline handles the complete build process for the **UAT environment**.

```bash
pipeline {
    agent any

    tools {
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage('CODE') {
            steps {
                git "https://github.com/PasupuletiBhavya/devsecops-project.git"
            }
        }

        stage('CQA') {
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

        stage('QualityGates') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
            }
        }

        stage('NPM Test') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t bhavyap007/finalround:UAT-v1 .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image bhavyap007/finalround:UAT-v1'
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker push bhavyap007/finalround:UAT-v1'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'
            slackSend (
                channel: 'all-camp',
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \nBuild: ${env.BUILD_NUMBER} \nDetails: ${env.BUILD_URL}"
            )
        }
    }
}
```

BUILD THIS PIPELINE→After successful build open SonarQube to check for bugs or vulnerabilities

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749685085634/fcbfec58-ac77-468b-8ee7-2db73bd7d589.png align="center")

## **Step-6: Integrating Jenkins with EKS Pre-Prod (UAT Namespace)**

#### 1\. **Service Account**

We first create a service account named `jenkins` in the `uat` namespace. This service account will be used by Jenkins to authenticate with the Kubernetes API.

#### 2\. **Role**

We then create a `Role` which defines **what actions** are allowed inside the namespace. For example:

* View pods, services, and deployments
    
* Create or delete resources
    
* Watch and update existing configurations
    

#### 3\. **RoleBinding**

The `RoleBinding` connects the **jenkins service account** with the defined **role**, granting the necessary access.

To **deploy into the EKS\_CLOUD (Pre-Prod) cluster** from Jenkins, we must first set up access and permissions using a **ServiceAccount** in the Kubernetes cluster.

#### Create a Directory for Manifests

```bash
mkdir manifests
cd manifests
```

#### Create a `service-account.yml` file

```bash
# service-account.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: uat
```

#### Apply the Manifest

```bash
kubectl create namespace uat
kubectl apply -f service-account.yml
#To verify that your Jenkins ServiceAccount was created successfully in the uat namespace, run:
kubectl get serviceaccount -n uat
```

This creates a **ServiceAccount named** `jenkins` inside the `uat` namespace of your EKS cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749685991179/8d872ce7-87c8-4de9-ac8e-bf4e6f8a3534.png align="center")

**Create role.yml file**

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: uat
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
      - secrets
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

**Create a rolebinding.yml file**

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: uat
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: uat
```

Run:

```bash
kubectl create -f role.yml
kubectl create -f rolebinding.yml
```

### **Creating a** [**Token**](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.) **for Jenkins ServiceAccount**

To allow Jenkins to authenticate and access the Kubernetes cluster, follow these steps to generate and retrieve the token:

**Create a secret YAML file called** `secret.yml`**:**

```bash
tapiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

> **Note:** Make sure the `jenkins` name matches your service account name.

**Apply the Secret in the** `uat` **Namespace**

```bash
kubectl apply -f secret.yml -n uat
```

> ⚠️ Since the namespace is not mentioned inside the YAML, you must pass it using `-n uat`.

**Retrieve the Token**

```bash
#Once created, run:
kubectl describe secret mysecretname -n uat
```

In the output, you’ll see a field called `token:` — **copy that value** and store it securely.

***Where to Use It?*** *You’ll use this token in Jenkins (via Kubernetes plugin) to authenticate and deploy workloads to the* `uat` *namespace.*

* **Open Jenkins** → Go to your pipeline job → Click **Pipeline Syntax** (bottom of the config page).
    
* In the dropdown, select:  
    `Kubernetes CLI Plugin` → `Configure Kubernetes CLI`
    
* **Add Credentials**:
    
    * Click on **“Add”** → Select **“Jenkins”**
        
    * Choose **“Secret text”**
        
    * Paste the **token** you copied from `kubectl describe secret mysecretname -n uat`
        
    * Set an **ID** like `k8-token`
        
    * Click **Add**
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749687941468/ad87d35f-3d02-4155-b0a0-8f82e33afc0d.png align="center")

Create new item → UAT\_DEPLOY\_PIPELINE  
(Kubernetes API endpoint you can find from your cluster dashboard)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749688015397/fa3185ad-309a-4e50-9970-c19f91497f87.png align="center")

```bash
pipeline {
    agent any
    environment {
        NAMESPACE = "uat" // Make sure to define the namespace if not passed as a parameter
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/PasupuletiBhavya/devsecops-project.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '', 
                    clusterName: 'EKS_CLOUD', 
                    contextName: 'myapp', 
                    credentialsId: 'k8s-token', 
                    namespace: "${NAMESPACE}", 
                    serverUrl: 'https://5D345FCC81F067526748BA123E5956EF.gr7.us-east-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl apply -f Manifests -n ${NAMESPACE}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '', 
                    clusterName: 'EKS_CLOUD', 
                    contextName: 'myapp', 
                    credentialsId: 'k8s-token', 
                    namespace: "${NAMESPACE}", 
                    serverUrl: 'https://5D345FCC81F067526748BA123E5956EF.gr7.us-east-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get all -n ${NAMESPACE}"
                    sh 'sleep 30'
                }
            }
        }
    }

    post {
        always {
            echo 'Sending Slack Notification...'
            slackSend (
                channel: 'all-camp',
                message: "*${currentBuild.currentResult}:* Job `${env.JOB_NAME}`\nBuild `${env.BUILD_NUMBER}`\nMore info: ${env.BUILD_URL}"
            )
        }
    }
}
```

BUILD PIPELINE AND ACCESS THE APPLICATION

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749688494407/b1df77af-f0c1-42a9-85a4-bfccc60f3f08.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749688526729/401734cf-7040-43ac-9eb9-385fa995b565.png align="center")

<mark>SO what we did so far</mark> ↓

![](https://sdmntprsouthcentralus.oaiusercontent.com/files/00000000-fe68-61f7-9325-ac755cc2d26c/raw?se=2025-06-12T01%3A44%3A46Z&sp=r&sv=2024-08-04&sr=b&scid=fdd76858-5130-5825-ae2d-f06441f2ca74&skoid=c953efd6-2ae8-41b4-a6d6-34b1475ac07c&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-06-11T11%3A06%3A48Z&ske=2025-06-12T11%3A06%3A48Z&sks=b&skv=2024-08-04&sig=%2BqQyHkCwh%2BlU5LoKL3PeTmHEI%2B9NVmFjdx9HPEwXzzY%3D align="left")

| **Jenkins EC2 does** | **Worker Node EC2 does** |
| --- | --- |

<table><tbody><tr><td colspan="1" rowspan="1"><p>Runs pipeline stages (build, scan, push)</p></td><td colspan="1" rowspan="1"><p>Actually runs your application pods</p></td></tr></tbody></table>

<table><tbody><tr><td colspan="1" rowspan="1"><p>Talks to Kubernetes via <code>kubectl</code></p></td><td colspan="1" rowspan="1"><p>Receives pods from Kubernetes scheduler</p></td></tr></tbody></table>

<table><tbody><tr><td colspan="1" rowspan="1"><p>Needs Kubernetes token to access the cluster</p></td><td colspan="1" rowspan="1"><p>Doesn’t need to know anything about Jenkins</p></td></tr></tbody></table>

<mark>Just follow the </mark> **<mark>same UAT process</mark>** <mark>inside the </mark> `staging` <mark>workspace. Only change: </mark> **<mark>Update namespace from </mark>** `uat` <mark>to </mark> `staging` <mark>wherever used.</mark>

Create a new JOBs ↓

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690099550/c4a3498b-4316-4281-9a3c-50dc98038094.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690150659/6879cc2c-b870-4676-b746-34ecfe7676f1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690169893/5f17c68b-8184-4287-82a8-8dae269b1f92.png align="center")

The **Build Pipeline** (`Stage_Build_Pipeline`) for staging is same as UAT — only the **Docker image tag** is updated to:

```bash
bhavyap007/finalround:staging-v1
```

The **Deploy Pipeline** (`Stage_Deploy_Pipeline`) for staging is also identical — just update:

* `namespace: "staging"`
    
* `credentialsId: "k8s-staging-token"`
    

All manifests will now apply in the `staging` namespace of `EKS_CLOUD`.  
*<mark>Make sure to update image in the dss.yml file in GitHub</mark>*

**BUILD the pipelines**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690714133/592526a3-cb9e-4b1d-a641-fc34b2a19dad.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690724415/f8c86299-50f3-4165-81a2-854e9c7bd24e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690934609/54755afc-140a-445b-8cf2-74e0875e41ab.png align="center")

Everything is updates in our slack

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749690914676/8cbc5204-5cbf-4bf6-bab6-a0764d458670.png align="center")

## **Step-7 : Now that staging is done, we create a new Kubernetes cluster just for production.**

create a new workspace prod

We switched to the `prod` workspace and applied the Terraform config to launch our **dedicated production EKS cluster**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749691419495/8ca563ae-abcb-4d64-9cbc-097078f83ff8.png align="center")

```bash
#We then update the kubeconfig:
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD_PROD
```

```bash
#The cluster is successfully connected — confirmed with:
kubectl get nodes
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749691771100/2cef3af6-d651-46c5-a029-5ba030a1f22d.png align="center")

create new namespace prod→ follow same steps as pre-prod

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749691660443/c101ebcc-63e9-461f-a82f-607c2f224671.png align="center")

1. Go to **Jenkins &gt; New Item**
    
2. Name it: `prod-deploy-pipeline`
    
3. In the **“Copy from”** field, enter: `Stage_Deploy_pipeline`
    
    **Now Just Update the Following:**
    
4. **Namespace** → `"prod"`
    
5. **credentialsId** → Your production token ID (e.g., `k8s-prod-token`)
    
6. **serverUrl** → Your `EKS_CLOUD_PROD` cluster endpoint URL
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749692159891/989679b0-5662-4290-8d0c-c21fc02471d9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749692172764/acac7578-bfc4-4b16-9eeb-786af1e45e86.png align="center")

Access application

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749692196745/8e48348a-b00c-48ca-987e-759241fb2c3e.png align="center")

## **Step-8: Install Argo CD Using Helm**

### Why Argo CD?

We use **Argo CD** to simplify and automate Kubernetes deployments using the **GitOps model**.

Instead of manually pushing changes with `kubectl`, Argo CD:

* Watches the Git repo for changes
    
* Automatically **pulls and syncs** updates to the cluster
    
* Gives a **dashboard** to monitor app status, rollout progress, and rollbacks
    

✅ More visibility  
✅ Better control  
✅ Production-grade delivery with **Git as the source of truth**

#### **Install Helm 3**

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version  # Confirm installation
```

#### **Add Argo CD Helm Repository**

```bash
helm repo add argo-cd https://argoproj.github.io/argo-helm
helm repo update
```

#### **Create Argo CD Namespace**

```bash
kubectl create namespace argocd
```

#### **Install Argo CD Using Helm**

```bash
helm install argocd argo-cd/argo-cd -n argocd
```

#### **Check Argo CD Resources**

```bash
kubectl get all -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765058024/f99f25c4-4f73-4bdb-bdf4-45b153499db9.png align="center")

### Accessing Argo CD in the Browser

By default, Argo CD is not exposed to the internet. To access its dashboard, we need to expose it externally using a **LoadBalancer** type service.

#### Convert ClusterIP to LoadBalancer

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

🔹 This changes the Argo CD service type from `ClusterIP` (internal) to `LoadBalancer` (public), so you can access it from your browser.

#### Install `jq` (JSON parser)

```bash
yum install jq -y
```

🔹 We use `jq` to extract the external hostname from the Argo CD service response easily.

#### Get the External Load Balancer URL

```bash
kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname
```

🔹 This command fetches the **public URL** (hostname) created by AWS ELB for your Argo CD service.

**Getting the Argo CD Admin Password**

When Argo CD is installed, it automatically creates a default admin account. The password is stored in a Kubernetes secret named `argocd-initial-admin-secret`.

```bash
# Export the admin password to a variable
export ARGO_PWD='kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d'
```

🔹 This command fetches the base64-encoded password from the Kubernetes secret and decodes it.  
🔹 It saves the command **as a string** to the `ARGO_PWD` variable — useful for reference.

> ⚠️ Note: This doesn't run the command. It just stores the text string.

#### Run the actual command to get the password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

🔹 This will **display the actual Argo CD admin password** in your terminal.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765546522/a5bfab54-44c1-4f99-9dfb-6cf526a6e7e6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765564593/0a96df5e-1d99-4656-b21b-0801e710a1f9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765577943/ff969a24-3408-48cd-8934-1ee6005c2149.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765589158/2e032876-b5e1-499a-8dcc-244c09164c6f.png align="center")

Argo CD gives you a **visual tree view** of your Kubernetes resources:

* Root node: the **application (**`camo-app`)
    
* Followed by:
    
    * `yelp-camp-service` (Service)
        
    * `yelp-camp-deployment` (Deployment)
        
    * ReplicaSet
        
    * 2 Pods — **both running & healthy**
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765689574/d3f93cda-78e6-4860-bc0a-4cc58a140997.png align="center")

If i make changes in my Git code suppose I change the number of replicas set to 4 agroCD will automatically pull the changes and sync it

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765905357/6cfba5fa-5120-42b2-a828-d6bebd4830f5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765916938/c9973da5-a3f7-476c-acc9-d48c208fc0f5.png align="center")

I can also update the image and deploy a different application

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765947743/34a7b0ff-74a9-4e90-8f6a-a782c266edef.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749765968018/47f31ac5-e547-4b60-b5a3-2da8d35115e5.png align="center")

You can see the application is changed from <mark>Camp</mark> to <mark>ZOMATO</mark>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749766021711/c0b5cc28-c7ed-468a-aae3-9434527aaef5.png align="center")

You can also rollback to previous camp version

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749766059658/4637225f-c657-4f91-bc90-2ad560e1c300.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749766080023/0f651229-68c5-48a9-9058-178620397522.png align="center")

## **Step-9: Now Last part is to delete resources to avoid billing**

```bash
# Do both for pre-prod and prod clusters
terraform destroy --auto-approve
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
terrraform workspace select default
terraform destroy --auto-approve
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1749766598712/0f5aaa44-22b7-41af-adfd-63d8c5b58e27.png align="center")

### Final Kubernetes Deployment Architecture

We use **two separate EKS clusters** to isolate staging/testing from actual production:

**Pre-Prod Cluster: EKS\_CLOUD**

* Hosts both **UAT** and **Staging** environments.
    
* Each environment is separated using **Kubernetes namespaces**.
    
* We build and test Docker images here.
    
* Once validated in Staging, the **same image** is promoted to Production.
    

**Production Cluster: EKS\_CLOUD\_PROD**

* Dedicated only for **Production deployment**.
    
* Ensures complete isolation from testing activities.
    
* Final deployment is done **without rebuilding**, directly using the verified image from staging.
    

This separation ensures:

* Stability and security in production.
    
* Easy troubleshooting and rollback in pre-prod without affecting users.
    

## What I Learned

This project gave me hands-on experience in building and deploying a secure, production-grade DevSecOps pipeline. Here's a quick summary:

**Application Setup**

* Built a 3-tier Node.js app (Yelp Camp) with Cloudinary & Mapbox integrations
    
* Used environment variables for secure config
    
* Hosted source code on GitHub
    

**Docker & CI/CD**

* Containerized the app using a lightweight Alpine image
    
* Scanned Docker images with Trivy
    
* Automated CI/CD using Jenkins pipelines (build, scan, push, deploy)
    

**Infrastructure as Code**

* Used Terraform to provision EC2 servers & EKS clusters
    
* Stored Terraform state securely in S3
    

**Multi-Stage Pipelines**

* Dev: Local testing & quick feedback
    
* UAT & Staging: Full build + deploy pipelines with Kubernetes integration
    
* Prod: GitOps-based deployment using Argo CD
    

**Kubernetes & GitOps**

* Created namespaces, roles, and bindings
    
* Jenkins connected to K8s with ServiceAccounts
    
* Argo CD auto-synced from Git and supported rollback
    

**Monitoring**

* Slack integrated for build/deploy notifications
    
* Argo CD UI helped track deployment status in real-time
    

---

For anyone starting out in DevOps, building a pipeline like this is one of the best ways to gain **practical, resume-worthy experience**.

If this article helped you in any way, your support would mean a lot to me 💕 — only if it's within your means.

Let’s stay connected on [**LinkedIn**](https://www.linkedin.com/in/bhavya-pasupuleti/) and grow together!

💬 Feel free to **comment or connect** if you have questions, feedback, or want to collaborate on similar projects.