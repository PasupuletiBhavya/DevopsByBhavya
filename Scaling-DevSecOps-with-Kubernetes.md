# Scaling DevSecOps with Kubernetes: Zero Downtime, Auto Healing, and More

## Introduction

In this third phase of our DevSecOps journey, we move from Docker-only deployment to a robust Kubernetes setup. Kubernetes enables:

* Zero Downtime during updates
* Rollbacks on failure
* Auto Scaling
* Self-Healing containers

We’ll cover:

* Kubernetes Basics
* Deployment Strategies
* EKS Setup on AWS
* Writing YAMLs
* Final Production Push
* GitOps with Argo CD
* Environment Isolation (UAT, Staging, Prod)

---

## Step 1: Launch EC2 Instance for Jenkins and Tool Installation

This EC2 instance acts as the **Ops Server**. It hosts your CI/CD tools like Jenkins and Trivy, and serves as the central control node that interacts with your Kubernetes cluster.

### EC2 Configuration:

* **AMI**: Amazon Linux 2 (Kernel 5.10 preferred for compatibility)
* **Instance Type**: `t2.large` (2 vCPUs, 8 GB RAM)
* **Storage**: Minimum 25 GB EBS volume
* **Security Group**: Allow inbound SSH (port 22)
* **Key Pair**: Choose an existing one or create a new key pair for SSH access
* **IAM Role**: Attach a role with access to EC2, S3, and EKS. You can reuse the role used in the Docker-based setup or create a fresh one.

Once the instance is launched, SSH into it using:

```bash
ssh -i your-key.pem ec2-user@<your-ec2-public-ip>
```

---

## Step 2: Install DevSecOps Toolchain

All commands below are to be executed on your EC2 Ops Server.

### Git (Version Control)

```bash
yum install git -y
```

###  Jenkins (CI Server)

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install java-17-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins at: `http://<your-ec2-public-ip>:8080`

###  Docker (Container Engine)

```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo chmod 777 /var/run/docker.sock
```

###  Terraform (IaC Tool)

```bash
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y
```

###  SonarQube (Code Quality Scanner)

Run it in a container:

```bash
docker run -itd --name sonar -p 9000:9000 sonarqube:lts-community
```

Access at: `http://<your-ec2-public-ip>:9000`

###  Trivy (Image Vulnerability Scanner)

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
```

###  AWS CLI (v2)

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

###  kubectl (Kubernetes CLI)

```bash
curl -LO "https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

###  eksctl (EKS Bootstrap Tool)

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" -o eksctl.tar.gz
tar -zxvf eksctl.tar.gz
sudo mv eksctl /usr/local/bin/
eksctl version
```

---

## Step 3: Provision an EKS Cluster Using Terraform

We’ll now create an EKS cluster using infrastructure-as-code principles with Terraform.

### 🔗 Clone Project Repo

```bash
git clone https://github.com/PasupuletiBhavya/devsecops-project.git
cd devsecops-project/k8s-project/eks-terraform
```

### 📁 File Breakdown:

* `backend.tf`: Defines remote state storage (S3 + DynamoDB)
* `provider.tf`: AWS provider configuration
* `main.tf`: Defines VPC, subnets, EKS cluster, node group, IAM roles, and security groups

### ⚙️ Initialize and Apply Terraform

```bash
terraform init               # Initialize the backend and download providers
terraform plan               # Dry run to preview changes
terraform apply --auto-approve   # Launch EKS cluster and related resources
```

Once finished, the cluster and worker nodes will be live in your AWS account.

---

## Step 4: Connect Jenkins EC2 Instance to EKS Cluster

After EKS creation, connect your EC2 Ops Server to the EKS cluster.

### ✅ Check Cluster Visibility

```bash
eksctl get cluster --region us-east-1
```

> Note: If `EKSCTL CREATED` shows as `False`, that's expected since Terraform provisioned it.

### 🔧 Configure kubectl Context

```bash
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
```

This sets your kubeconfig to use the new cluster.

### 🔍 Verify Connection to Cluster

```bash
kubectl get nodes
```

You should see a list of worker nodes in the `Ready` state.

---

✅ You’ve now completed the foundational setup. Jenkins is installed, your Kubernetes cluster is live, and connectivity is confirmed. The next phase involves creating Jenkins pipelines that build, scan, push, and deploy your application to this EKS cluster.


## Step 5: CI/CD for Staging & Production

### Staging Build Pipeline

Same as UAT pipeline, update image tag to `bhavyap007/finalround:staging-v1`

### Staging Deploy Pipeline (Jenkinsfile)

Update namespace to `staging` and credentials ID accordingly:

```groovy
credentialsId: 'k8s-staging-token'
namespace: 'staging'
```

### Production Deploy Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any
    environment {
        NAMESPACE = "prod"
    }
    stages {
        stage('Deploy to Prod') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS_CLOUD_PROD',
                    contextName: 'prod-app',
                    credentialsId: 'k8s-prod-token',
                    namespace: "${NAMESPACE}",
                    serverUrl: 'https://<prod-eks-endpoint>'
                ]]) {
                    sh "kubectl apply -f Manifests -n ${NAMESPACE}"
                }
            }
        }
    }
    post {
        always {
            slackSend (
                channel: 'all-camp',
                message: "*${currentBuild.currentResult}:* Job `${env.JOB_NAME}`\nBuild `${env.BUILD_NUMBER}`\nMore info: ${env.BUILD_URL}"
            )
        }
    }
}
```

---

## Step 6: Install and Configure Argo CD (GitOps)

### Install Helm 3 and Argo CD

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm repo add argo-cd https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo-cd/argo-cd -n argocd
```

### Expose Argo CD

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname
```

### Get Argo CD Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## Step 7: Using GitOps with Argo CD

Once connected, Argo CD can:

* Auto sync deployments from GitHub
* Watch manifests for changes (e.g., replicas, image updates)
* Visualize app structure and pod status
* Support manual or auto rollback

### GitOps Flow:

1. Update GitHub manifests (e.g., replicas to 4)
2. Argo CD syncs changes to cluster automatically
3. Argo dashboard shows live rollout

---

## Step 8: Cleanup Resources

### Destroy Pre-Prod and Prod Resources

```bash
terraform destroy --auto-approve
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
terraform workspace select default
terraform destroy --auto-approve
```

---

## Final Kubernetes Deployment Architecture

### EKS\_CLOUD (Pre-Prod)

* Namespaces: `uat`, `staging`
* Used for full CI/CD and validation

### EKS\_CLOUD\_PROD (Production)

* Isolated from testing
* GitOps powered with Argo CD
* Uses same verified Docker image from staging

---

## What I Learned

* Real-world CI/CD with Jenkins, Docker, Kubernetes
* GitOps with Argo CD for production stability
* Infrastructure-as-Code with Terraform
* Image Scanning with Trivy, Quality checks with SonarQube
* Slack notifications and complete deployment visibility

---

✅ Project Source: [GitHub Repo](https://github.com/PasupuletiBhavya/devsecops-project)
🙌 Connect with me on LinkedIn for feedback, questions, or DevSecOps collaboration!
