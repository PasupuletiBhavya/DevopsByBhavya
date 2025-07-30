# 🚀 Campus Drive Management: A Complete DevSecOps Journey (6 Phases Explained)

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1749501564851/014c7ca4-8e42-407e-8c28-1447fc91d686.png)

This project simulates a real-world enterprise application rollout using a complete **DevSecOps pipeline** — from development to production deployment, including automation, security, and infrastructure-as-code.

---

## 🧭 Phase 1: Understanding Client Requirements & Local Development

Developers collect business needs to build a **Campus Drive Management System** that allows:

* Scheduling drives
* Uploading images and pricing info
* Tracking metro city-wise events

They build the **frontend**, **backend**, and **database** components and test the app locally. Source code is pushed to **GitHub**.

---

## 🌩️ Phase 2: Cloud Infra Provisioning Using Terraform

Rather than manual provisioning, we use **Terraform** to define and deploy EC2 instances with:

* CPU, RAM, Storage specs
* VPC/Subnet/network configuration

Terraform enables repeatable, version-controlled infrastructure deployment.

---

## 🐳 Phase 3: Dockerization & Security Scanning

### Steps:

1. Create `Dockerfile`
2. Build image
3. Run container locally
4. **Scan image with Trivy** for vulnerabilities
5. Push image to Docker Hub
6. Deploy container on EC2 to test behavior

---

## 🔁 Phase 4: CI/CD Automation with Jenkins (DevSecOps Enabled)

A Jenkins pipeline automates the following:

1. GitHub checkout
2. SonarQube code analysis
3. Docker build
4. Trivy scan
5. Push to Docker Hub
6. Deploy container on EC2
7. Send Slack alert

We use a **Jenkins Master-Slave model**: Jenkins Master orchestrates jobs, slaves (EC2) execute them.

---

## 🚦 Phase 5: Multi-Stage Pipelines (Dev, Test, Staging, Prod)

### 🧪 Dev Pipeline:

* Runs on Dev EC2
* Unit testing only

### ✅ Testing Pipeline:

* Runs on Testing EC2
* Runs UAT, Regression tests

### 🚀 Staging & Production Pipeline:

* Uses **Terraform** to provision servers
* Final approval-based production deploy
* Uses same Docker image throughout
* Slack/Email for status updates

---

## ☸️ Phase 6: Production-Grade Deployment with Kubernetes

We switch from Docker-only to Kubernetes for:

* Auto-scaling
* Zero downtime via rolling updates
* Self-healing pods
* Namespace isolation

Apps are deployed via `kubectl` using YAML manifests into a live cluster.

---

## 🧩 Why Docker First?

* Easier to learn and debug
* Reusable image for Kubernetes
* Faster feedback loop

## 🧱 Why Kubernetes Later?

* Needed for scale, HA, and enterprise needs
* Supports health checks, rollbacks, auto-scaling

---

## 🔗 Related Blogs

👉 [**Docker CI/CD DevSecOps Blog**](https://devops-by-bhavya.hashnode.dev/cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment)

👉 [**Kubernetes Production Setup Blog**](https://devops-by-bhavya.hashnode.dev/scaling-devsecops-with-kubernetes-zero-downtime-auto-healing-and-more)

---

## ✅ Final Outcome

You now have a real-world **DevSecOps pipeline** integrating:

* ✅ Terraform (infra-as-code)
* ✅ Docker + Trivy (container + security)
* ✅ Jenkins (CI/CD)
* ✅ SonarQube (code quality)
* ✅ Kubernetes (production-grade deploys)

Perfect for portfolios, resumes, or real-world use!

---

📬 Let’s stay connected: [**LinkedIn – Bhavya Pasupuleti**](https://www.linkedin.com/in/bhavya-pasupuleti/)

💬 Comment or connect if you have feedback or want to collaborate!
