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

## \[Steps 1–4: EC2 Launch, Stack Installation, EKS with Terraform, Jenkins Setup]

(Same as before — see previous section for detailed bash and YAML code)

---

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
