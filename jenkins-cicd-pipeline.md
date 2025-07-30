# End-to-End CI/CD Pipeline with Jenkins on AWS

**Using Git, Maven, SonarQube, Nexus, and Tomcat to Automate Code Build, Test, and Deployment**

## 🛠️ Tech Stack
- **Git**: Source code management
- **Maven**: Build tool
- **SonarQube**: Code quality scanner
- **Nexus**: Artifact repository
- **Tomcat**: Application server
- **Jenkins**: CI/CD automation
- **AWS EC2**: Infrastructure for hosting services

---

## 🚀 Step-by-Step Setup

### Step 1: Launch EC2 Instances
Launch **4 EC2 instances** using the same PEM file:
- Jenkins: t2.micro
- Tomcat: t2.micro
- SonarQube: t2.medium + 25GB EBS
- Nexus: t2.medium + 25GB EBS

Set up each tool on its own instance.

### Step 2: Install DevOps Tools

**Jenkins**
```bash
yum install git -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo yum install fontconfig java-21-openjdk jenkins
sudo systemctl daemon-reload
yum install java-17-amazon-corretto -y
yum install jenkins -y
systemctl start jenkins.service
```

**Tomcat**
```bash
yum install java-17-amazon-corretto -y
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.104/bin/apache-tomcat-9.0.104.tar.gz
tar -zxvf apache-tomcat-9.0.104.tar.gz
# Add users
# Remove IP restrictions
sh bin/startup.sh
```

**Nexus**
```bash
yum install java-17-amazon-corretto -y
cd /opt
wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz
tar -zxvf nexus-unix-x86-64-3.79.0-09.tar.gz
useradd nexus
# Edit nexus run_as_user
su - nexus
/opt/nexus-3.79.0-09/bin/nexus start
```

**SonarQube**
```bash
amazon-linux-extras install java-openjdk11 -y
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
su - sonar
cd /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/
sh sonar.sh start
```

---

### Step 3: Configure Jenkins

- Install Plugins:
  - SonarQube Scanner
  - Nexus Artifact Uploader
  - SSH Agent

- Add Maven installer (v3.9.9)

---

### Step 4: Jenkinsfile Pipeline

```groovy
node {
    stage("Code") {
        git "https://github.com/PasupuletiBhavya/one.git"
    }

    stage("Build") {
        def mavenHome = tool name: "maven3", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

    stage("CQA") {
        withSonarQubeEnv('mysonar') {
            def mavenHome = tool name: "maven3", type: "maven"
            def mavenCMD = "${mavenHome}/bin/mvn"
            sh "${mavenCMD} sonar:sonar"
        }
    }

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

    stage("Deployment") {
        sshagent(['8bce0b0e-c5db-4aac-bf07-4831ca13a760']) {
            sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@52.23.240.218:/home/ec2-user/apache-tomcat-9.0.104/webapps/'
        }
    }
}
```

---

## 🧠 What I Learned

- Integrated Jenkins with SonarQube, Nexus, Tomcat.
- Automated build-test-deploy workflow.
- Managed AWS EC2 infrastructure.
- Hands-on DevOps implementation experience.

---

## 🔗 Resources

- GitHub: [https://github.com/PasupuletiBhavya](https://github.com/PasupuletiBhavya)
- Jenkins: https://www.jenkins.io
- SonarQube: https://www.sonarqube.org
- Nexus: https://www.sonatype.com
- Tomcat: https://tomcat.apache.org

---

💡 _If this helped you, consider connecting on [LinkedIn](https://linkedin.com/in/bhavya-pasupuleti)._ 
