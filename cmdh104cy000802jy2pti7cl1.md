---
title: "Full-Stack Observability on AWS EKS: Prometheus, Grafana & ELK with Helm"
seoTitle: "Full-Stack Observability on AWS EKS with Prometheus, Grafana & ELK"
seoDescription: "Learn how to implement full-stack observability on AWS EKS using Prometheus, Grafana, and ELK stack with Helm in this comprehensive guide"
datePublished: Thu Jul 24 2025 06:43:10 GMT+0000 (Coordinated Universal Time)
cuid: cmdh104cy000802jy2pti7cl1
slug: full-stack-observability-on-aws-eks-prometheus-grafana-and-elk-with-helm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1753339159080/917838db-40b8-47f0-9c77-54d7a17294f8.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1753339283164/228b1027-70fb-4b8c-85ee-670dc79296ba.png
tags: kibana, aws, kubernetes, monitoring, elasticsearch, elk, helm, prometheus, filebeat, grafana, eks, observability

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753337606809/c5dcf6eb-d01b-4718-ba50-b005bf5aee3b.png align="center")

## **Introduction**

In the world of DevOps, **monitoring and logging are non-negotiable essentials**. They provide real-time visibility into system health, help debug issues faster, and ensure proactive performance tuning. Without robust observability, even the most resilient applications can fail silently.

For this full-day hands-on lab, my goal was to **build a complete monitoring and logging pipeline** on Kubernetes using open-source tools. I wanted to gain end-to-end visibility into pod performance, node resource usage, and container logs—all while hosting everything on **AWS EKS**.

To achieve this, I used the following stack:

* **Prometheus** – for collecting and storing metrics
    
* **Grafana** – for visualizing and alerting on metrics
    
* **Elasticsearch** – for indexing and storing logs
    
* **Kibana** – for log visualization and analytics
    
* **Filebeat** – to ship Kubernetes logs to Elasticsearch
    
* **Helm** – to simplify deployments of all components
    
* **EKS (Elastic Kubernetes Service)** – as the managed Kubernetes platform
    

This blog walks through the exact steps I followed—from cluster setup to visualizing logs and metrics—and the key takeaways from this practical observability journey.

## Step 1:Setting Up the EKS Admin EC2 Instance To interact with the Kubernetes cluster on EKS

I first launched an EC2 instance that serves as my administration node. From this machine, I installed and used tools like kubectl, eksctl, and helm.

✅ EC2 Instance Configuration:

| Field | Value |
| --- | --- |
| Name | eks-admin-ec2 |
| AMI | Amazon Linux 2 (x86\_64) |
| Instance Type | t2.medium (or t3.medium if not using free tier) |
| Key Pair | Created/Used existing key pair (e.g., eks-key) |
| Network | Default VPC selected |
| Security Group | Allowed SSH (22), HTTP (80), HTTPS (443) |
| Storage | 20 GiB (gp3) |

This instance ran in the us-east-1a availability zone and used a public IPv4 address for easy access.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753311855334/ac2186c6-2529-486a-b92f-8f680016883b.png align="center")

## Phase 1: Connect and Initial Setup

Once the EC2 admin instance was running, I SSH’d into it and installed all necessary tools to interact with my EKS cluster.

### Step 1: Update the System

```bash
sudo yum update -y
```

### Step 2: Install Required Tools

#### ✅ Install Docker

```bash
sudo yum install docker -y
sudo systemctl enable docker
sudo systemctl start docker
```

#### ✅ Install `kubectl`

```bash
curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/
kubectl version --client
```

#### ✅ Install `eksctl`

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz
sudo mv eksctl /usr/local/bin
```

#### ✅ Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

#### ✅ Install `jq`

```bash
sudo yum install jq -y
```

### Step 3: Install AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### Step 4: Configure AWS CLI

```bash
aws configure
```

You’ll be prompted to enter:

* **AWS Access Key ID**
    
* **AWS Secret Access Key**
    
* **Default region name:** `us-east-1`
    
* **Default output format:** `json`
    

## Phase 2: Creating the EKS Cluster using `eksctl`

With all the required tools installed and the AWS CLI configured, I used `eksctl` to spin up a fully managed EKS cluster on AWS.

### 🚀 Cluster Creation Command

```bash
eksctl create cluster \
  --name devops-cluster \
  --region us-east-1 \
  --nodes 2 \
  --node-type t3.medium \
  --with-oidc \
  --managed
```

### What this command does:

* `--name devops-cluster`: Names the cluster.
    
* `--region us-east-1`: Deploys the cluster in N. Virginia.
    
* `--nodes 2`: Starts with 2 worker nodes.
    
* `--node-type t3.medium`: Each node uses `t3.medium` instance type.
    
* `--with-oidc`: Enables OIDC provider for IAM roles for service accounts (IRSA).
    
* `--managed`: Uses AWS-managed node groups for easier upgrades and scaling.
    

The provisioning process automatically:

✅ Creates the EKS control plane  
✅ Sets up a VPC (if not provided)  
✅ Deploys managed worker nodes  
✅ Configures Kubernetes add-ons like `coredns`, `kube-proxy`, and `metrics-server`

After around 15 minutes, the cluster and its node group were fully ready to use.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753313649807/fac0342b-a944-4b6d-9347-c707bff14d7b.png align="center")

## Verifying the EKS Cluster Nodes

```bash
kubectl get nodes
#Check the service and get the external IP (LoadBalancer):
kubectl get svc
```

This confirmed that both nodes were in `Ready` state and running Kubernetes version `v1.32.3-eks`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753313981236/8aeab86f-5da2-42b2-8fef-4c2dc60b467f.png align="center")

### Deploying My Application to EKS

After creating the EKS cluster, I deployed my application using Kubernetes manifests directly from my GitHub repository. This helped me generate live traffic and logs, which were later used for observability.

I ran the following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/PasupuletiBhavya/devsecops-project/master/Manifests/dss.yml
```

This deployed my application (YelpCamp-style app) to the cluster, making it accessible via a LoadBalancer. With this running app, I could proceed to set up Prometheus, Grafana, and the ELK stack to monitor logs and metrics.

*Test the Load Balancer URL*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753314164980/59510d7f-c50b-4d0c-90b5-fa3a18e791f7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753314188303/48a00414-75e3-434f-822f-8ef9f10c8cb3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753314200651/b72dcccc-196b-4119-aac1-e8ace737ac4a.png align="center")

## Phase 3: Monitoring with Prometheus & Grafana

With the EKS cluster ready, I moved on to deploying a comprehensive monitoring solution using the `kube-prometheus-stack` Helm chart, which bundles Prometheus, Grafana, and several Kubernetes observability tools.

### Step 1: Add Helm Repo for Prometheus Community Charts

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Step 2: Install `kube-prometheus-stack` Chart

```bash
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace
```

This installs a complete monitoring suite, including:

* ✅ **Prometheus** – for metrics collection
    
* ✅ **Grafana** – for dashboards and visualizations
    
* ✅ **Alertmanager** – for alert handling
    
* ✅ **Node Exporter** – to expose node metrics
    
* ✅ **Kube State Metrics** – for Kubernetes object monitoring
    

### Step 3: Validate Deployment

I checked the pods and services in the `monitoring` namespace:

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

All components were up and running:

#### ✅ Pods Status

| Pod Name | Status |
| --- | --- |
| `kube-prom-stack-grafana` | Running |
| `kube-prom-stack-kube-prome-prometheus` | Running |
| `kube-state-metrics` | Running |
| `node-exporter` | Running |
| `alertmanager` | Running |

## Phase 4: Exposing Grafana & Prometheus via LoadBalancer

By default, the Grafana and Prometheus services are internal-only (ClusterIP). To access their UIs from the browser, I patched both services to be of type **LoadBalancer**.

### Step 1: Patch Services

#### 🔄 Expose Grafana:

```bash
kubectl patch svc kube-prom-stack-grafana -n monitoring \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

#### 🔄 Expose Prometheus:

```bash
kubectl patch svc kube-prom-stack-kube-prome-prometheus -n monitoring \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

### Step 2: Get External IPs

After patching, I waited a minute and ran:

```bash
kubectl get svc -n monitoring
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316088248/bf836d57-9b5a-4ee7-b703-009d09b359c4.png align="center")

This returned external DNS endpoints for both Grafana and Prometheus:

| **Service** | **External IP / URL** |
| --- | --- |
| `kube-prom-stack-grafana` | [`a865c57d922fe4de4b08cef4e3c0aeee-2006643144.us-east-1.elb.amazonaws.com`](http://a865c57d922fe4de4b08cef4e3c0aeee-2006643144.us-east-1.elb.amazonaws.com) (Port 80) |
| `kube-prom-stack-kube-prome-prometheus` | [`a8a9d4669a7144b5caeddf178c5e8eab-21081589.us-east-1.elb.amazonaws.com`](http://a8a9d4669a7144b5caeddf178c5e8eab-21081589.us-east-1.elb.amazonaws.com) (Port 9090) |

> 📌 *You can now access:*
> 
> * **Grafana:** `http://<grafana-external-ip>`
>     
> * **Prometheus:** `http://<prometheus-external-ip>`
>     

### Step 3: Retrieve Grafana Admin Password

To log in to the Grafana dashboard, I fetched the auto-generated admin password:

```bash
kubectl get secret kube-prom-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

**Credentials:**

* **Username:** `admin`
    
* **Password:** *(output from the above command)*
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753315783701/3f3e0971-3882-44f3-9f41-1323129fa94f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753315808117/f3876350-95c3-4cac-a39a-7ed3e1a31809.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753315828179/c7133498-a841-4d0c-b8c5-7b082b261983.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753315854336/80b5b189-47bb-4216-9d8b-3300ee167796.png align="center")

#### 👉 Steps to Import:

1. Go to **Grafana UI**.
    
2. In the left sidebar, click **“+” ➝ Import**.
    
3. Enter the dashboard ID (e.g., `1860`) and click **Load**.
    
4. Select Prometheus as the data source and click **Import**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753315909356/fb5cce22-a2d7-4097-b498-b11650eb6562.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316178021/c2c1f6df-f3eb-4aa9-9f58-1fe3866b52b7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316194968/fe060748-542c-464a-be8f-2389f86319b1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316208688/ab171299-ff03-4196-a611-72f945eedca0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316240769/b5941c17-a0b8-4fbe-8e46-3a1fa4a9133e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316255077/891b8ba3-f197-457a-b6fd-b7b111fff50c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316278234/a89df493-2bba-4f0a-82dd-a6bbe23acc46.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316331224/190328a3-8b2b-49be-acb9-4906a240c0c1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753316342952/4eaaf2d2-bf41-4b6d-9c8a-084c99403f92.png align="center")

## Phase 6: External Endpoint Monitoring with Blackbox Exporter

In real-world production systems, it’s essential to monitor not just internal metrics but also the **availability of external endpoints**. For this, I used **Blackbox Exporter** with Prometheus to actively probe the HTTP status of my deployed app.

### 🚀 Step 1: Define a `Probe` Resource

I created a custom `Probe` object using the following YAML to monitor my external application running at port `3000`:

```bash
apiVersion: monitoring.coreos.com/v1
kind: Probe
metadata:
  name: campground-probe
  namespace: monitoring
  labels:
    probe: "true"   # ✅ Required for Prometheus to scrape this probe
spec:
  jobName: "blackbox-campground"
  interval: 30s
  module: http_2xx
  prober:
    url: blackbox-exporter.monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
        - http://a6df26611d1c84f4d9431caf2ebe7e1f-1142985076.us-east-1.elb.amazonaws.com:3000/
```

This config tells Prometheus to:

* Check the URL every 30 seconds
    
* Use `http_2xx` module to verify HTTP status
    
* Use Blackbox Exporter inside the `monitoring` namespace
    

### ✅ Result: Probe is UP! 🟢

Prometheus successfully picked up the probe, and I could see it turn **green (UP)** in the Prometheus Targets UI. This means:

* ✅ Prometheus is scraping the probe endpoint
    
* ✅ Blackbox Exporter is reachable
    
* ✅ The external app is up and responding correctly
    

### Visualize Probe Status in Grafana

You can visualize Blackbox status directly in Grafana by creating a panel with the following PromQL:

```bash
probe_success{job="blackbox-campground"}
```

This will show a **1 (UP)** or **0 (DOWN)** based on the latest probe result.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753317359334/6839766e-d0f5-43e8-b199-058fde1de493.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753317395097/aef8eb5a-f0c6-4ac6-9888-ed3e9e51c9b5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753317405185/4013b681-0a9a-406a-ba65-c6c3ca21da4e.png align="center")

## Phase 7: Deploying ELK Stack for Kubernetes Log Monitoring

While Prometheus and Grafana give us great metrics visibility, we also need to monitor **application and system logs**. That’s where the **ELK stack**—Elasticsearch, Logstash (optional), and Kibana—comes in. I deployed the ELK stack using Helm charts for simplicity.

### Step 1: Add Elastic Helm Repository

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

### Step 2: Create Namespace for ELK

```bash
kubectl create namespace elk
```

### Step 3: Install Elasticsearch

```bash
helm install elasticsearch elastic/elasticsearch \
  -n elk \
  --set volumeClaimTemplate.storageClassName=gp2 \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set resources.requests.memory=512Mi \
  --set resources.requests.cpu=100m \
  --set resources.limits.memory=1Gi \
  --set resources.limits.cpu=500m
```

> ⚠️ I initially faced an issue where the Elasticsearch pod was stuck in `Pending` due to EBS volume provisioning failure. The fix was to:
> 
> * **Install the AWS EBS CSI driver**
>     
> * **Attach the** `AmazonEBSCSIDriverPolicy` to my worker node IAM role
>     
> * Then reattempt deployment
>     

You can check pod status using:

```bash
kubectl get pods -n elk -l app=elasticsearch-master
```

### Step 4: Install Kibana

```bash
helm install kibana elastic/kibana -n elk \
  --set service.type=LoadBalancer
```

Then get the external IP:

```bash
kubectl get svc -n elk | grep kibana
```

Access Kibana at:

```bash
http://<EXTERNAL-IP>:5601
```

### Step 5: Install Filebeat (Log Forwarder)

```bash
helm install filebeat elastic/filebeat -n elk \
  --set daemonset.enabled=true \
  --set elasticsearch.hosts="{http://elasticsearch-master.elk.svc.cluster.local:9200}"
```

Check Filebeat pods:

```bash
kubectl get pods -n elk -l app=filebeat
```

### Step 6: Access Logs in Kibana

Once Kibana is up:

1. Open `http://<EXTERNAL-IP>:5601`
    
2. Go to **“Discover”**
    
3. You should start seeing Kubernetes logs (collected by Filebeat and indexed by Elasticsearch)
    

### What Each Component Does:

| Component | Role |
| --- | --- |
| **Elasticsearch** | Stores the logs indexed from the cluster |
| **Filebeat** | Collects logs from all Kubernetes nodes |
| **Kibana** | Provides a dashboard for visualizing and searching logs |

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753318946582/ec4ddcc5-896a-4e3c-8bd7-830b6b53210a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753318973668/15e9a319-288f-426a-8dc7-0eafa9e6f793.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753319022154/9331784c-abd4-4823-823d-dfef6551a6fa.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753319040071/d40ae506-e084-477d-a08a-9bee127b7653.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753319066170/9366b292-bc97-4af4-ab3b-c334d5194fa9.png align="center")

### ✅ Elasticsearch is Up and Running!

After deploying the Elasticsearch Helm chart and patching the service to `LoadBalancer`, I accessed it via:

```bash
http://<elasticsearch-external-ip>:9200
```

As seen in the screenshot:

```bash
{
  "name": "elasticsearch-master-0",
  "cluster_name": "elasticsearch",
  "version": {
    "number": "8.5.1",
    ...
  },
  "tagline": "You Know, for Search"
}
```

This confirms that:

* Elasticsearch is accessible
    
* Cluster health is good
    
* Ready to receive logs from Filebeat
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753324386600/80c432fe-9149-40ba-a11d-8e50e4375d67.png align="center")

### What This Confirms:

* ✅ Your **Elasticsearch Pod is running**
    
* ✅ The LoadBalancer service is **working externally**
    
* ✅ Elasticsearch is **responding correctly** on port `9200`
    
* ✅ You have **secure HTTPS access** to the API
    

## Elasticsearch Setup and External Access (via Load Balancer)

### ✅ What I Achieved:

* Deployed **Elasticsearch** on **Amazon EKS** using Helm
    
* Exposed Elasticsearch service using **Load Balancer** (not just port-forward)
    
* Verified secure external access using the default `elastic` user credentials
    

### Steps I Followed:

#### 1\. **Check Pod Status**

```bash
kubectl get pods -n elk
```

✅ `elasticsearch-master-0` was in `Running` state with `1/1` containers ready.

#### 2\. **Get Elasticsearch Password**

```bash
kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

🔐 Password: `c6WWaP7tt26OGiY8`

#### 3\. **Expose Elasticsearch via LoadBalancer**

```bash
kubectl patch svc elasticsearch-master -n elk \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

#### 4\. **Get External Access URL**

```bash
kubectl get svc -n elk
```

🔗 URL:

```bash
https://ab42505e742ba4deab140d74087ed823-28402472.us-east-1.elb.amazonaws.com:9200
```

#### 5\. **Verify from Browser / cURL**

```bash
curl -u elastic:c6WWaP7tt26OGiY8 -k https://ab42505e742ba4deab140d74087ed823-28402472.us-east-1.elb.amazonaws.com:9200
```

### 📘 Notes:

* I used **self-signed certs**, hence `-k` flag for cURL
    
* Elastic Helm chart defaults to **TLS enabled**, which is ideal for production
    

## Step-by-Step: Deploying Kibana & Verifying ELK Stack Access on Kubernetes (with Load Balancer)

After successfully exposing **Elasticsearch**, the next phase was to install **Kibana** and make it externally accessible via a LoadBalancer. Here's how I did it:

### ✅ Step 1: Install Kibana via Helm with LoadBalancer Enabled

```bash
helm install kibana elastic/kibana -n elk \
  --set service.type=LoadBalancer \
  --set elasticsearchHosts=https://elasticsearch-master:9200 \
  --set resources.requests.memory=512Mi \
  --set resources.requests.cpu=100m \
  --set resources.limits.memory=1Gi \
  --set resources.limits.cpu=500m
```

📝 Notes:

* `elasticsearchHosts` points to your Elasticsearch service (inside cluster).
    
* Resource limits help manage memory/CPU in Kubernetes.
    

### Step 2: Wait for External IP

After a few minutes, I fetched the external IP:

```bash
kubectl get svc -n elk
```

✔️ I saw the **EXTERNAL-IP** assigned to `kibana-kibana` like:

```bash
kibana-kibana   LoadBalancer   10.100.x.x   abcd1234.elb.amazonaws.com   5601:xxxx/TCP
```

### Step 3: Access the Kibana UI

Open your browser:

```bash
http://<EXTERNAL-IP>:5601
```

🧭 You’ll land on the **Kibana dashboard**, ready to visualize logs and metrics.

### (Optional) Configure Public Base URL for Kibana

To avoid redirect issues, especially when accessing Kibana behind a LoadBalancer:

```bash
--set kibanaConfig.kibana.yml.server.publicBaseUrl=http://<EXTERNAL-IP>:5601
```

---

### 💡 What’s Next?

Now that both **Elasticsearch** and **Kibana** are publicly accessible, you can:

* Explore Elasticsearch data in Kibana
    
* Create index patterns and dashboards
    
* Add Filebeat or Logstash to ingest logs
    
* Combine with Grafana dashboards or Prometheus alerts
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753324860566/3ad5936b-d7ae-4957-9fb0-b4133b5616e0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753324874968/a0bbc14b-031c-430c-8ae1-d219aed1537f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753324886274/4ea11160-ef8e-486e-9f6c-cc086325b807.png align="center")

## Filebeat Setup for Real-Time Log Shipping

To send Kubernetes pod logs to Elasticsearch, I used **Filebeat + Autodiscover**.

### ✅ Add Elastic Helm repo :

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

### ✅ Created `filebeat-values.yaml` for Autodiscover

```bash
filebeatConfig:
  filebeat.yml: |
    filebeat.autodiscover:
      providers:
        - type: kubernetes
          hints.enabled: true
          hints.default_config:
            type: container
            paths:
              - /var/log/containers/*.log
    output.elasticsearch:
      hosts: ["https://elasticsearch-master:9200"]
      username: "elastic"
      password: "c6WWaP7tt26OGiY8"
      ssl.verification_mode: "none"
```

Saved as `filebeat-values.yaml`

### Install Filebeat

```bash
helm install filebeat elastic/filebeat -n elk \
  -f filebeat-values.yaml
```

Checked status:

```bash
kubectl get pods -n elk
```

✅ Filebeat pod was running!

### Verifying Logs in Kibana

Once Filebeat was running:

* Opened Kibana
    
* **Create an Index Pattern**:
    
    * Go to **"Stack Management" → "Index Patterns"**
        
    * Click **"Create index pattern"**
        
    * Enter:
        
        ```bash
        filebeat-*
        ```
        
    * Select `@timestamp` as the time filter field
        
    * Click **Create index pattern**
        
* **Go to Discover Tab**:
    
    * Navigate to **“Discover”**
        
    * Select your new index pattern (`filebeat-*`)
        
    * You should see logs coming in from your containers!
        

🎉 I could see real-time logs from Kubernetes pods being shipped to Elasticsearch via Filebeat!

## ✅ Final Setup Overview

| Component | Status |
| --- | --- |
| Elasticsearch | ✅ Running (LB) |
| Kibana | ✅ Running (LB) |
| Filebeat | ✅ Installed |
| Logs in Kibana | ✅ Verified |

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332437771/7bdc0277-22d4-4171-9bae-d0f89c98dd92.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332480399/32d494e0-e314-4b32-8044-264771d004dc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332498775/1e6e1dad-ea30-4cf0-a0a0-9deb52f7761c.png align="center")

## Live Kubernetes Logs in Kibana Discover

After successfully installing Filebeat and connecting it to Elasticsearch, I moved to **Kibana's Discover tab** to view live logs.

### 📸 Here’s a snapshot of my Kibana dashboard:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332622896/9a513f3e-d96b-4926-8752-a4ca067ffdf7.png align="center")

As you can see:

* I queried the `filebeat-*` index
    
* Kibana showed **live pod logs** from my Kubernetes cluster
    
* The logs contain metadata like:
    
    * `agent.hostname`: which Filebeat pod shipped the logs
        
    * `kubernetes.namespace`: `elk`
        
    * [`container.name`](http://container.name): which container the log came from
        
    * `timestamp`, `node`, `zone`, `topology`, and more
        

🎉 **Success:** My entire EKS cluster logs are now searchable and filterable in Kibana.

### What’s Happening Behind the Scenes:

* Filebeat is running as a DaemonSet in Kubernetes
    
* It autodetects containers using Kubernetes hints
    
* Logs from `/var/log/containers/*.log` are shipped securely to Elasticsearch
    
* Kibana visualizes and indexes them for querying
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332723239/ac779da4-1d48-49a2-9fed-457f97208508.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332753970/74cbaaa3-7356-4be3-8b9a-0f6f1e8b5ded.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753332779227/08c22f86-2d93-4926-80af-c98cd909a793.png align="center")

## Create Your First Visualization

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333196191/cab48f8e-c950-4b2b-bb4f-345151829efc.png align="center")

After setting up Filebeat and confirming logs were reaching Elasticsearch, I used **Kibana Lens** to quickly visualize log volume.

### What I Did:

* Selected the `filebeat-*` index
    
* Used `@timestamp` on the X-axis
    
* Set Y-axis to show the **count of records**
    
* Time range: Last 5 minutes
    
* Chart type: Vertical bar
    

### What It Shows:

This chart displays how many logs are coming in every few seconds. It helps confirm:

* Filebeat is sending logs continuously
    
* There are no big gaps or sudden spikes
    

Simple and effective way to monitor log flow in real time. ✅

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333390707/99282122-e53f-4c22-b862-928779af8740.png align="center")

## 🍩 Pod-wise Log Distribution (Donut Chart)

To understand which pods are generating the most logs, I created a **donut chart** in Kibana.

### What I Did:

* Index pattern: `filebeat-*`
    
* Slice by: Top 5 values of [`kubernetes.pod.name`](http://kubernetes.pod.name)
    
* Size by: **Count of records**
    
* Time range: Last 5 minutes
    

### What It Shows:

This chart shows the log volume share from each pod.

For example:

* `filebeat-filebeat-77fs9` generated 25% of logs
    
* `elasticsearch-master-0` and `grafana` each contributed ~21%
    
* Helps spot noisy pods or identify issues quickly
    

A great way to **visually monitor log load** across components!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333464161/87af99f3-dd51-43f4-9f60-4b5db86b54e9.png align="center")

## Log Distribution by Pod – Latest View

To track how logs are distributed among the top 5 Kubernetes pods, I generated this donut chart.

### Setup:

* **Index pattern**: `filebeat-*`
    
* **Slice by**: Top 5 values of [`kubernetes.pod.name`](http://kubernetes.pod.name)
    
* **Size by**: Count of records
    

### Insights:

* `filebeat-filebeat-77fs9` contributed the most logs (~37%)
    
* `elasticsearch-master-0` and `filebeat-filebeat-wg9ws` each logged ~31%
    
* This helps quickly identify the busiest pods in terms of logging
    

👉 Great for spotting potential log flooding or heavy activity.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333574566/40fb7257-f342-4e59-a957-4a582bd7c627.png align="center")

## Pod-Wise Log Distribution – Bar Chart View

To visualize log volume by pod, I created a vertical bar chart using Filebeat data in Kibana.

### Configuration:

* **Index pattern**: `filebeat-*`
    
* **X-axis**: Top 5 values of [`kubernetes.pod.name`](http://kubernetes.pod.name)
    
* **Y-axis**: Count of log records
    

### Quick Takeaway:

* `filebeat-filebeat-77fs9` generated the highest number of logs.
    
* Other pods like `filebeat-filebeat-wg9ws` and `elasticsearch-master-0` also show steady activity.
    
* This helps in quickly identifying which pods are generating the most logs in near real time.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333750028/dec1cc3a-d9bd-4c9b-bf73-999580e834d1.png align="center")

## Namespace-Wise Log Activity Over Time

This bar chart shows how log events are distributed across Kubernetes namespaces (`elk`, `kube-system`, and `monitoring`) over time.

### Configuration:

* **Index pattern**: `filebeat-*`
    
* **X-axis**: `@timestamp` (interval: 30 seconds)
    
* **Y-axis**: Unique count of `kubernetes.namespace_labels.kubernetes_io/metadata_name`
    
* **Breakdown**: Top 3 values of `kubernetes.namespace`
    

### Insights:

* Most activity came from the `elk` namespace, which includes Elasticsearch and Kibana.
    
* `kube-system` and `monitoring` show consistent but lower activity.
    
* This helps verify if logs are being collected from all critical namespaces.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333798877/a6fb0e38-04a5-470d-b798-e6069b8b26d5.png align="center")

## Log Count per Namespace Over Time

This visualization shows log traffic trends in the `elk` and `monitoring` namespaces over the past 15 minutes.

### Configuration:

* **Index pattern**: `filebeat-*`
    
* **X-axis**: `@timestamp` (interval: 30 seconds)
    
* **Y-axis**: `Count of records`
    
* **Breakdown**: Top 3 values of `kubernetes.namespace`
    

### Key Observations:

* The `elk` namespace is consistently generating logs, which makes sense as it runs Elasticsearch and Kibana.
    
* The `monitoring` namespace shows periodic spikes, indicating bursts of log activity, possibly from Prometheus or Grafana.
    

This breakdown helps validate that Filebeat is capturing logs across key namespaces as expected.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333922343/12c13d3f-4c83-4658-83ad-564fbaffb96e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753333985068/45744c3b-857a-4422-954b-445ed5b3a335.png align="center")

## Error and Failure Logs Tracked Over Time

This chart filters and visualizes logs that include error or failure indicators such as:

```bash
"error" OR log.level = "error" OR "ERR" OR "failed"
```

### Configuration:

* **Index pattern**: `filebeat-*`
    
* **X-axis**: `@timestamp` (30-minute intervals)
    
* **Y-axis**: `Count of records`
    
* **Breakdown**: `Top 3 values of` [`kubernetes.container.name`](http://kubernetes.container.name)
    

### Observation:

* A spike in error-related logs was observed from the **Grafana** container during the recent time window.
    
* This helps proactively identify which services are experiencing issues and when.
    

✅ This kind of filtering and visualization is essential for real-time troubleshooting and alerting.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753334069315/35674120-b8eb-4d74-9f20-1ebdfb5165ba.png align="center")

## Final Cleanup (No Billing Left)

### Deleted Cluster

```bash
eksctl delete cluster --name devops-cluster --region us-east-1
```

### Manually Cleaned:

* Helm releases
    
* EBS volumes (via EC2 dashboard)
    
* Load balancers & security groups
    
* IAM roles
    
* CloudWatch log groups
    
    ### What I Did: E2E Kubernetes Observability Stack in a Day
    
    ### ✅ Why Monitoring & Logging Matter
    
    In DevOps, real-time observability is critical for:
    
    * Detecting issues before users do
        
    * Troubleshooting failures quickly
        
    * Analyzing system health & resource usage
        
    
    ### '🗓️ My Goal
    
    Build a **complete monitoring + logging setup on Kubernetes** using open-source tools and clean it up to avoid billing.
    
    **Tools Used**
    
    * **Amazon EKS** – Kubernetes cluster on AWS
        
    * **Helm** – Easy deployment of complex apps
        
    * **Prometheus + Grafana** – Metrics monitoring & dashboards
        
    * **Elasticsearch + Kibana + Filebeat (ELK)** – Centralized log aggregation & analysis
        
        ### **Final Thoughts**
        
        Setting up an end-to-end observability stack on Kubernetes might seem overwhelming at first — but with the right tools and a structured approach, it becomes manageable and rewarding.
        
        This hands-on exercise helped me:
        
        * Understand how logs and metrics flow in real-world clusters
            
        * Troubleshoot Helm installation issues
            
        * Visualize logs using Kibana and monitor metrics via Grafana
            
        * Clean up infrastructure to avoid unnecessary AWS billing
            
        
        Whether you're learning DevOps or managing production-grade clusters, observability is a skill worth mastering.
        
        ### 🔗 Reference
        
        I referred to this excellent guide during my setup process:  
        [**Comprehensive AWS EKS Cluster Monitoring with Prometheus, Grafana, and EFK Stack – 10 Weeks of CloudOps**  
        ](https://devo.hashnode.dev/comprehensive-aws-eks-cluster-monitoring-with-prometheus-grafanaand-efk-stack-10weeksofcloudops)A big thanks to the author for such a well-structured walkthrough!
        
        **Thanks for reading!**  
        If you found this helpful[,](https://devo.hashnode.dev/comprehensive-aws-eks-cluster-monitoring-with-prometheus-grafanaand-efk-stack-10weeksofcloudops) feel free to connect with me or drop your thoughts in the comments.