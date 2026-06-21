# 🚀 Production-Ready Blue-Green Deployment Using Jenkins & AWS ALB

## 📖 Project Overview

This project demonstrates a complete production-grade Blue-Green Deployment strategy using AWS, Jenkins, GitHub, and Application Load Balancer (ALB).

The objective is to eliminate deployment downtime by maintaining two identical environments:

* 🔵 Blue Environment (Currently Live)
* 🟢 Green Environment (Standby)

Instead of deploying directly to production servers, new versions are deployed to the inactive environment. After successful validation and health checks, traffic is switched automatically through the Application Load Balancer.

If validation fails, the deployment is rolled back automatically without affecting end users.

---

# 🎯 Objectives

* Maintain two separate environments
* Deploy updates without affecting production traffic
* Perform automated health checks
* Switch traffic only after successful validation
* Enable automatic rollback
* Achieve zero downtime deployments

---

# 🏗️ Architecture

```text
Users
   │
   ▼
Application Load Balancer
(blue-green-alb)
   │
 ┌─┴───────────────┐
 │                 │
 ▼                 ▼

Blue Server      Green Server
Version 1        Version 2

   ▲
   │
 Jenkins Server
   │
   ▼
GitHub Repository
```

---

# 🛠️ Tech Stack

| Service    | Purpose                       |
| ---------- | ----------------------------- |
| AWS EC2    | Blue, Green & Jenkins Servers |
| AWS ALB    | Traffic Routing               |
| Jenkins    | CI/CD Pipeline                |
| GitHub     | Source Control                |
| Nginx      | Web Server                    |
| AWS CLI    | ALB Management                |
| IAM        | AWS Permissions               |
| Ubuntu     | Operating System              |
| OpenJDK 21 | Jenkins Runtime               |

---

# 📋 Prerequisites

Before starting:

* AWS Account
* GitHub Account
* EC2 Key Pair (.pem)
* AWS CLI
* Ubuntu EC2 Instances
* Internet Access

---

# Step 1: Launch EC2 Infrastructure

Create three EC2 instances.

## Blue Server

Configuration:

```text
Name: blue-server
OS: Ubuntu LTS
Instance Type: t3.micro
```

Inbound Rules:

```text
SSH (22) → My IP
SSH (22) → Jenkins Private IP
HTTP (80) → Anywhere
```

---

## Green Server

Configuration:

```text
Name: green-server
OS: Ubuntu LTS
Instance Type: t3.micro
```

Inbound Rules:

```text
SSH (22) → My IP
SSH (22) → Jenkins Private IP
HTTP (80) → Anywhere
```

---

## Jenkins Server

Configuration:

```text
Name: jenkins-server
OS: Ubuntu LTS
Instance Type: t3.micro
```

Inbound Rules:

```text
SSH (22) → My IP
HTTP (80) → Anywhere
TCP 8080 → Anywhere
```

---

# Step 2: Connect to Blue & Green Servers

```bash
ssh -i your-key.pem ubuntu@<blue-public-ip>
```

```bash
ssh -i your-key.pem ubuntu@<green-public-ip>
```

---

# Step 3: Install Nginx

Run on BOTH servers.

```bash
sudo apt update -y
```

```bash
sudo apt install -y nginx
```

```bash
sudo systemctl start nginx
```

```bash
sudo systemctl enable nginx
```

Verify:

```bash
sudo systemctl status nginx
```

Expected:

```text
active (running)
```

---

# Step 4: Deploy Initial Application

## Blue Server

```bash
echo "<h1>Its blue-server!</h1>" | sudo tee /var/www/html/index.html
```

```bash
echo "OK" | sudo tee /var/www/html/health
```

---

## Green Server

```bash
echo "<h1>Its green-server!</h1>" | sudo tee /var/www/html/index.html
```

```bash
echo "OK" | sudo tee /var/www/html/health
```

---

# Step 5: Verify Web Servers

Open:

```text
http://<blue-public-ip>
```

Expected:

```text
Its blue-server!
```

Open:

```text
http://<green-public-ip>
```

Expected:

```text
Its green-server!
```

---

# Step 6: Create Target Groups

## Blue Target Group

AWS Console:

```text
EC2 → Target Groups
```

Configuration:

```text
Name: blue-tg
Target Type: Instances
Protocol: HTTP
Port: 80
Health Check Path: /health
```

Register:

```text
blue-server
```

---

## Green Target Group

Configuration:

```text
Name: green-tg
Protocol: HTTP
Port: 80
Health Check Path: /health
```

Register:

```text
green-server
```

---

# Step 7: Create Application Load Balancer

Navigate:

```text
EC2 → Load Balancers
```

Configuration:

```text
Name: blue-green-alb
Scheme: Internet Facing
Listener: HTTP :80
```

Select:

```text
At least 2 Availability Zones
```

Security Group:

```text
HTTP (80) → 0.0.0.0/0
```

---

# Step 8: Configure ALB Listener

Default Action:

```text
Forward To → blue-tg
```

This makes Blue Environment live.

Wait:

```text
1-2 Minutes
```

ALB Status:

```text
Active
```

Test:

```text
http://<alb-dns>
```

Expected:

```text
Its blue-server!
```

---

# Step 9: Install Jenkins

SSH into Jenkins Server:

```bash
ssh -i your-key.pem ubuntu@<jenkins-public-ip>
```

Update:

```bash
sudo apt update -y
```

Install Java:

```bash
sudo apt install openjdk-21-jre-headless -y
sudo apt install openjdk-21-jdk-headless -y
```

Install Nginx:

```bash
sudo apt install nginx -y
```

Install AWS CLI:

```bash
sudo apt install unzip -y
```

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```bash
unzip awscliv2.zip
```

```bash
sudo ./aws/install
```

Verify:

```bash
aws --version
```

Install Jenkins:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

```bash
sudo apt update
```

```bash
sudo apt install jenkins -y
```

```bash
sudo systemctl start jenkins
```

```bash
sudo systemctl enable jenkins
```

---

# Step 10: Unlock Jenkins

Open:

```text
http://<jenkins-public-ip>:8080
```

Get Password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Paste password.

Install Suggested Plugins.

Create Admin User.

---

# Step 11: Install Jenkins Plugins

Install:

```text
Pipeline
Git Plugin
SSH Agent
```

Restart Jenkins:

```bash
sudo service jenkins restart
```

---

# Step 12: Configure SSH Access

Copy PEM:

```bash
scp -i your-key.pem your-key.pem ubuntu@<jenkins-public-ip>:/home/ubuntu/
```

Move Key:

```bash
sudo mkdir -p /var/lib/jenkins/.ssh
```

```bash
sudo cp /home/ubuntu/your-key.pem /var/lib/jenkins/.ssh/
```

```bash
sudo chown jenkins:jenkins /var/lib/jenkins/.ssh/your-key.pem
```

```bash
sudo chmod 400 /var/lib/jenkins/.ssh/your-key.pem
```

---

# Step 13: Test SSH

```bash
sudo su - jenkins
```

```bash
touch ~/.ssh/known_hosts
```

Blue:

```bash
ssh -i ~/.ssh/your-key.pem ubuntu@<blue-private-ip>
```

Green:

```bash
ssh -i ~/.ssh/your-key.pem ubuntu@<green-private-ip>
```

---

# Step 14: Create GitHub Repository

Repository:

```text
blue-green-deploy
```

Structure:

```text
blue-green-deploy/
│
├── app
│   ├── index.html
│   └── health
│
└── Jenkinsfile
```

---

# Step 15: Push Code

```bash
git init
```

```bash
git add .
```

```bash
git commit -m "Initial blue green deployment"
```

```bash
git branch -M main
```

```bash
git remote add origin https://github.com/<username>/blue-green-deploy.git
```

```bash
git push -u origin main
```

---

# Step 16: Create Jenkins Pipeline

```text
New Item
```

Name:

```text
blue-green-pipeline
```

Type:

```text
Pipeline
```

SCM:

```text
Git
```

Branch:

```text
*/main
```

---

# Step 17: Configure AWS CLI For Jenkins

```bash
sudo su - jenkins
```

```bash
aws configure
```

Enter:

```text
Access Key
Secret Key
Region
Output Format
```

Test:

```bash
aws elbv2 describe-load-balancers
```

---

# Step 18: Deploy New Version

Pipeline Flow:

```text
Pick Target
↓
Deploy
↓
Health Check
↓
Switch ALB Traffic
```

---

# Step 19: Health Check Logic

Pipeline checks:

```bash
curl -f http://<target-server>/health
```

Expected:

```text
OK
```

---

# Step 20: Automatic Traffic Switching

If health check succeeds:

```bash
aws elbv2 modify-listener
```

Traffic moves:

```text
Blue → Green
```

or

```text
Green → Blue
```

---

# Step 21: Automatic Rollback

If health check fails:

```text
ALB listener restored to previous Target Group
```

Result:

```text
Users continue using old environment
No downtime occurs
```

---

# Step 22: Validation

Verify:

```bash
cat /var/www/html/index.html
```

```bash
cat /var/www/html/health
```

Check:

```bash
sudo systemctl status nginx
```

Open:

```text
http://<alb-dns>
```

Expected:

```text
Version 2 - Deployed by Jenkins
```

---

# 🎉 Final Outcome

Successfully implemented:

✅ Blue-Green Deployment

✅ Jenkins CI/CD

✅ Application Load Balancer

✅ Automated Health Checks

✅ Automatic Traffic Switching

✅ Automatic Rollback

✅ Zero Downtime Releases

---

# 🤝 Contributing

Fork the repository and create a feature branch.

Submit a Pull Request describing your changes and improvements.

---

# 📜 License

This project is intended for educational and DevOps learning purposes.
