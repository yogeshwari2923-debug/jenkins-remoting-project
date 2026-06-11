# Jenkins Remoting & Distributed Build System

A fully distributed CI/CD infrastructure built on AWS using Jenkins Remoting to connect and manage remote agent nodes securely.

## Project Overview

This project demonstrates how to set up Jenkins Remoting to distribute build loads across multiple AWS EC2 instances, run jobs on different architectures, and improve security using node isolation.

## Architecture
Developer
|
v
Jenkins Controller (EC2 - t3.medium)
|          |
v          v
Agent Node 1   Agent Node 2
(Linux x86)    (Linux ARM)

## Tech Stack

- **Jenkins** — CI/CD Automation Server
- **AWS EC2** — Cloud Infrastructure
- **Amazon Linux 2023** — Operating System
- **Java 21 (Amazon Corretto)** — Jenkins Runtime
- **systemd** — Agent Auto-start Service

## Infrastructure Setup

| Instance | Type | Role | OS |
|---|---|---|---|
| jenkins-controller | t3.medium | Jenkins Controller | Amazon Linux 2023 |
| jenkins-agent-1 | t3.micro | Build Agent (x86) | Amazon Linux 2023 |
| jenkins-agent-2 | t3.micro | Build Agent (ARM) | Amazon Linux 2023 |

## Security Configuration

| Port | Purpose | Access |
|---|---|---|
| 22 | SSH | My IP only |
| 8080 | Jenkins Web UI | 0.0.0.0/0 |
| 50000 | Agent Communication | VPC only |

## Project Implementation Steps

### Step 1 — Controller Setup
```bash
# Java 21 Install
sudo yum install java-21-amazon-corretto -y

# Jenkins Install
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import \
    https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### Step 2 — Agent Setup
```bash
# Java 21 Install
sudo yum install java-21-amazon-corretto -y

# Create working directory
mkdir -p /home/ec2-user/jenkins-agent
cd /home/ec2-user/jenkins-agent

# Download agent jar
curl -sO http://<controller-private-ip>:8080/jnlpJars/agent.jar

# Connect to Controller
java -jar agent.jar \
  -url http://<controller-private-ip>:8080/ \
  -secret <secret-token> \
  -name "agent-node-1" \
  -workDir "/home/ec2-user/jenkins-agent"
```

### Step 3 — Agent Auto-start Service
```bash
# /etc/systemd/system/jenkins-agent.service
[Unit]
Description=Jenkins Agent
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/jenkins-agent
ExecStart=/usr/bin/java -jar /home/ec2-user/jenkins-agent/agent.jar \
  -url http://<controller-private-ip>:8080/ \
  -secret <secret-token> \
  -name "agent-node-1" \
  -workDir "/home/ec2-user/jenkins-agent"
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Step 4 — Security Hardening
- Controller executors set to **0** — Controller never runs build code
- Port 50000 restricted to VPC only — Agents not exposed to internet
- Each agent runs in isolated environment

### Step 5 — Distributed Job Execution
```bash
# Job runs on specific agent using label
echo "Running on: $(hostname)"
echo "Architecture: $(uname -m)"
echo "Build Number: $BUILD_NUMBER"
```

## Key Features

- **Distributed Builds** — Jobs spread across multiple agents automatically
- **Node Isolation** — Controller only manages, never executes build code
- **Auto Recovery** — Agents auto-restart on failure using systemd
- **Multi Architecture** — Jobs run on x86 and ARM agents
- **Secure Communication** — Agents connect via secret token on port 50000

## Project Outcomes

- Successfully connected 2 remote agent nodes to Jenkins Controller
- Distributed build jobs across different EC2 instances
- Implemented node isolation for improved security
- Configured auto-start service for agents using systemd
- Ran jobs on multiple architectures remotely

## Screenshots

> Add your screenshots here
- Jenkins Dashboard
- Nodes Page (Both agents connected)
- Console Output (Build success on agent)
- AWS EC2 Instances

## What I Learned

- How Jenkins Remoting protocol works (Controller ↔ Agent communication)
- Distributed CI/CD systems used in production environments
- AWS EC2 infrastructure setup for DevOps workflows
- Security best practices in Jenkins (Node isolation, secret tokens)
- systemd service configuration for auto-start processes

## Author

**Dhineshkumar V (Yogi)**
AI & Full Stack Developer
Salem, Tamil Nadu, India

---
*Project completed as part of CodeAlpha Internship*
