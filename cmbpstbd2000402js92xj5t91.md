---
title: "Campus Drive Management: A Complete DevSecOps Journey (6 Phases Explained)"
seoTitle: "Project Flow + DevSecOps Overview"
seoDescription: "Project Flow + DevSecOps Overview"
datePublished: Tue Jun 10 2025 00:44:27 GMT+0000 (Coordinated Universal Time)
cuid: cmbpstbd2000402js92xj5t91
slug: campus-drive-management-a-complete-devsecops-journey-6-phases-explained
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1749501564851/014c7ca4-8e42-407e-8c28-1447fc91d686.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1749774741459/e5a2b4b8-e1b6-408a-b500-2bc81f7b15a9.png
tags: docker, github, deployment, sonarqube, kubernetes, slack, automation, devops, terraform, jenkins, devsecops, ci-cd, docker-images, devops-articles, trivy

---

This project simulates a real-world enterprise application rollout using a complete **DevSecOps pipeline** — from development to production deployment, including automation, security, and infrastructure-as-code.

Here’s a breakdown of how we built and deployed the system step by step.

## **Phase 1: Understanding Client Requirements & Development Setup**

We begin by gathering the client’s requirements. Suppose a client wants a **campus drive management application** to manage weekly drives across various metro cities. This application should allow the company to:

* Schedule drives
    
* Track city-wise events
    
* Upload images and pricing info
    

Once the requirements are clear, developers write the code for the **front-end**, **back-end**, and **database**, then push the source code to **GitHub**. Initially, the application is tested locally on their personal systems to validate core functionality.

## **Phase 2: Cloud Infrastructure Setup Using Terraform**

Since applications aren’t meant to run locally in production, we **migrate them to the cloud**☁️ . In this case, we use **AWS EC2** as our virtual server.

But instead of creating servers manually, we use **Terraform** to automate infrastructure creation. The developer provides specifications (e.g., 2 CPUs, 4 GB RAM, 25 GB storage), and the DevOps engineer writes Terraform code to spin up EC2 instances with those resources.

## **Phase 3: Application Containerization with Docker & Security Scanning**

Next, we **containerize the application** using **Docker** 🐳:

1. Write a `Dockerfile`
    
2. Build a Docker image from the source code
    
3. Run the image to launch a container
    

To ensure security, we scan the image using **Trivy**, which detects vulnerabilities in OS packages and dependencies. If the image is clean, we **push it to Docker Hub**.

Finally, the container is run on our **cloud-based EC2 server** to verify if it behaves the same way outside the local environment.

## **Phase 4: Automating the Pipeline with Jenkins (DevSecOps CI/CD)**

Now we automate all manual steps using a **CI/CD pipeline** written in **Jenkins**.

### Jenkins Pipeline Flow:

1. Pull Code from GitHub
    
2. Run SonarQube analysis for code quality
    
3. Build Docker image
    
4. Scan image with Trivy
    
5. Push to Docker Hub
    
6. Run the container
    
7. Send Slack notification
    

To run this pipeline **on the EC2 instance**, we configure a **Jenkins Master-Slave setup** — where Jenkins Master handles orchestration and EC2 (slave) executes the pipeline.

## **Phase 5: Multi-Stage Pipelines for Dev, Test, Staging & Prod**

Instead of a single pipeline, we create **three separate pipelines**, each targeting a specific environment:

### 1️⃣ Dev Pipeline

* **Purpose:** For developers to test newly written features.
    
* **Runs:** On the **Dev EC2 instance**.
    
* **Stages:**
    
    * Pull code from GitHub
        
    * Run SonarQube analysis
        
    * Build Docker image
        
    * Scan image with Trivy
        
    * Deploy to Dev environment
        
    * Notify via Slack
        
* ✅ **Only Unit Testing** happens here
    

### 2️⃣ Testing Pipeline

* **Purpose:** For QA teams to validate builds before client access.
    
* **Runs:** On a separate **Testing EC2 instance**
    
* **Stages:**
    
    * Pull code (optional) or reuse Dev build
        
    * Run Functional, UAT, Regression Tests
        
    * Deploy latest Docker image to testing
        
    * Notify team
        
* ✅ Ensures the build is functionally stable
    

### 3️⃣ Staging & Production Pipeline

* **Purpose:** Final pipeline for client approval and release
    
* **Runs:** On **Staging and Production servers**, created via **Terraform**
    
* **Stages:**
    
    * Deploy the approved image to Staging
        
    * Client verifies functionality
        
    * If approved, **same image** is deployed to Production
        
    * Notify stakeholders via Slack or Email
        
* ✅ Uses **Terraform** for infra provisioning
    
* ✅ No manual steps — completely automated
    

## **Phase 6: Production Deployment Using Kubernetes**

To ensure **zero downtime**, **scalability**, and **fault tolerance**, we use **Kubernetes (K8s)** for our production deployment.

### ❌ Drawbacks of Docker-only:

* Downtime during container restarts (10s–1min)
    
* No version rollback to arbitrary versions
    
* No built-in auto-scaling
    

### ✅ Kubernetes Solves This With:

* Auto-scaling based on load
    
* Rolling updates and easy rollbacks
    
* Self-healing containers
    
* Cluster-based deployment using master/worker nodes
    
* Namespace isolation for dev/test/staging separation
    

Using `kubectl` and deployment YAML files, we deploy the application into a **Kubernetes cluster**, achieving a **production-grade deployment flow**.

## Final Outcome

By the end of this project, we implemented a **real-world DevSecOps pipeline** that includes:

* Infrastructure as code with **Terraform**
    
* Containerization and image scanning using **Docker + Trivy**
    
* CI/CD automation via **Jenkins**
    
* Code quality assurance with **SonarQube**
    
* Scalable and secure deployment using **Kubernetes**
    

Whether you're working on a startup project or deploying at enterprise scale, this DevSecOps pipeline offers **speed, automation, and resilience** right from Day 1.

---

**Why Start with Docker First?**

1. **Simplicity & Clarity:**  
    Docker is easier to set up and helps you understand containerization basics — how images, containers, volumes, and networks work.
    
2. **Faster Feedback Loop:**  
    You can test changes quickly on your local machine or cloud VM without needing to configure an entire cluster.
    
3. **Build Once, Run Anywhere:**  
    Once your app is containerized properly with Docker, the same image can later be used in Kubernetes — **no need to rebuild**.
    
4. **Better Debugging:**  
    Easier to troubleshoot issues in Docker before introducing Kubernetes complexity.
    

**Why Move to Kubernetes Later?**

1. **Scale & Reliability:**  
    When you're ready to handle high availability, load balancing, and auto-scaling — Kubernetes becomes essential.
    
2. **Zero Downtime:**  
    K8s supports rolling updates, self-healing, and real-time rollback — all things Docker doesn’t handle natively.
    
3. **Real-world Readiness:**  
    Most companies use Docker **with** Kubernetes in production. So building Docker-first aligns well with long-term DevOps skill-building and deployment strategy.
    

## Related Blogs :

👉 Want to see only the Docker version in detail?  
[**Check out my Docker CI/CD Blog here**](https://devops-by-bhavya.hashnode.dev/cicd-pipeline-with-docker-devsecops-for-real-world-app-deployment)

👉 Want to dive into the Kubernetes production setup?  
[**Read my Kubernetes Blog here**](https://devops-by-bhavya.hashnode.dev/scaling-devsecops-with-kubernetes-zero-downtime-auto-healing-and-more)

---

For anyone starting out in DevOps, building a pipeline like this is one of the best ways to gain **practical, resume-worthy experience**.

If this article helped you in any way, your support would mean a lot to me 💕 — only if it's within your means.

Let’s stay connected on [**LinkedIn**](https://www.linkedin.com/in/bhavya-pasupuleti/) and grow together!

💬 Feel free to **comment or connect** if you have questions, feedback, or want to collaborate on similar projects.