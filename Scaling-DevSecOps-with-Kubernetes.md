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

---

## Step 1: Launch EC2 Instance

* **AMI**: Amazon Linux 2 (Kernel 5.10)
* **Instance Type**: t2.large
* **Storage**: 25 GB EBS
* **Key Pair**: Existing or new
* **IAM Role**: EC2, S3, EKS full access
* **Security Group**: Port 22 open

---

## Step 2: Install DevSecOps Tech Stack

### Git

```bash
yum install git -y
```

### Jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install java-17-amazon-corretto -y
yum install jenkins -y
systemctl start jenkins
systemctl enable jenkins
systemctl status jenkins
```

### Docker

```bash
yum install docker -y
systemctl start docker
systemctl enable docker
systemctl status docker
chmod 777 /var/run/docker.sock
```

### Terraform

```bash
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y
```

### SonarQube (Docker)

```bash
docker run -itd --name sonar -p 9000:9000 sonarqube:lts-community
```

### Trivy

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
```

### AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### kubectl

```bash
curl -LO "https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### eksctl

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" -o eksctl.tar.gz
tar -zxvf eksctl.tar.gz
sudo mv eksctl /usr/local/bin/
eksctl version
```

---

## Step 3: Provision EKS using Terraform

```bash
git clone https://github.com/PasupuletiBhavya/devsecops-project.git
cd k8s-project/eks-terraform
```

Update `backend.tf`, `provider.tf`, `main.tf` with your configuration.

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

---

## Step 4: Connect to EKS

```bash
eksctl get cluster --region us-east-1
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
kubectl get nodes
```

---

## Jenkins Server vs Worker Node

* **Jenkins EC2** = control center for pipeline automation
* **EKS Worker Node** = hosts app containers

---

## CI/CD Pipeline (UAT)

### Jenkinsfile (UAT Build)

```groovy
pipeline {
    agent any
    tools { nodejs 'node16' }
    environment { SCANNER_HOME = tool 'mysonar' }
    stages {
        stage('CODE') {
            steps { git "https://github.com/PasupuletiBhavya/devsecops-project.git" }
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
            steps { waitForQualityGate abortPipeline: false, credentialsId: 'sonar' }
        }
        stage('NPM Test') {
            steps { sh 'npm install' }
        }
        stage('Docker Build') {
            steps { sh 'docker build -t bhavyap007/finalround:UAT-v1 .' }
        }
        stage('Trivy Scan') {
            steps { sh 'trivy image bhavyap007/finalround:UAT-v1' }
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
            slackSend (
                channel: 'all-camp',
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \nBuild: ${env.BUILD_NUMBER} \nDetails: ${env.BUILD_URL}"
            )
        }
    }
}
```

---

## Kubernetes Manifests for Jenkins Deployment

### service-account.yml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: uat
```

### role.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: uat
rules:
  - apiGroups: ["", "apps", "autoscaling", "batch", "extensions", "policy", "rbac.authorization.k8s.io"]
    resources: ["pods", "services", "deployments", "secrets", "configmaps", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### rolebinding.yml

```yaml
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

### secret.yml

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

Apply all:

```bash
kubectl create namespace uat
kubectl apply -f service-account.yml
kubectl apply -f role.yml
kubectl apply -f rolebinding.yml
kubectl apply -f secret.yml -n uat
```

Retrieve token:

```bash
kubectl describe secret mysecretname -n uat
```

Use in Jenkins: Add `Secret Text` with ID: `k8-token`

---

# Scaling DevSecOps with Kubernetes: Zero Downtime, Auto Healing, and More (Part 2 of 2)

## Jenkins UAT Deploy Pipeline

```groovy
pipeline {
    agent any
    environment {
        NAMESPACE = "uat"
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
                    credentialsId: 'k8-token',
                    namespace: "${NAMESPACE}",
                    serverUrl: 'https://<eks-endpoint>'
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
                    credentialsId: 'k8-token',
                    namespace: "${NAMESPACE}",
                    serverUrl: 'https://<eks-endpoint>'
                ]]) {
                    sh "kubectl get all -n ${NAMESPACE}"
                    sh 'sleep 30'
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

## Staging Environment Pipelines

Create two jobs like UAT, but change:

* Docker image tag to:

```bash
bhavyap007/finalround:staging-v1
```

* Namespace to `staging`
* Kubernetes credentials ID to `k8s-staging-token`

Update manifests accordingly.

---

## Production Environment Setup

Create a new Terraform workspace:

```bash
terraform workspace new prod
terraform apply --auto-approve
```

Update kubeconfig:

```bash
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD_PROD
kubectl get nodes
```

Create `prod` namespace and Jenkins token:

```bash
kubectl create namespace prod
kubectl apply -f prod-manifests/
```

Create new Jenkins job by copying `Stage_Deploy_pipeline`. Update:

* Namespace → `prod`
* credentialsId → `k8s-prod-token`
* serverUrl → EKS\_CLOUD\_PROD URL

---

## Argo CD Setup using Helm

### Install Helm 3

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Add Argo CD Helm Repo

```bash
helm repo add argo-cd https://argoproj.github.io/argo-helm
helm repo update
```

### Create Namespace & Install Argo CD

```bash
kubectl create namespace argocd
helm install argocd argo-cd/argo-cd -n argocd
kubectl get all -n argocd
```

### Expose Argo CD via LoadBalancer

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### Get Argo CD External URL

```bash
yum install jq -y
kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname
```

### Get Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## GitOps with Argo CD

* Add Git repo with manifests
* Argo CD watches repo, syncs changes
* Visualize and rollback deployments easily

You can:

* Change replica count → Argo syncs
* Change image → New app rolls out
* Rollback → Previous version re-applies

---

## Clean Up Resources

```bash
terraform destroy --auto-approve
aws eks update-kubeconfig --region us-east-1 --name EKS_CLOUD
terraform workspace select default
terraform destroy --auto-approve
```

---

## Final Kubernetes Deployment Architecture

**Pre-Prod Cluster (EKS\_CLOUD):**

* Namespaces: `uat`, `staging`
* Full CI/CD: build → test → deploy

**Prod Cluster (EKS\_CLOUD\_PROD):**

* Isolated from staging
* Argo CD GitOps deploy only (no rebuilds)

---

## Summary of Learnings

* Dockerized 3-tier app (Yelp Camp)
* CI/CD with Jenkins, SonarQube, Trivy
* EKS cluster provisioning with Terraform
* Namespace & RBAC setup for Jenkins-to-K8s
* GitOps deployments via Argo CD
* Monitoring with Slack notifications

This hands-on project mirrors real-world DevSecOps practices, ready to showcase on resumes and GitHub.

---


