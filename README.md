# 🚀 Yii2 DevOps Assessment - Docker Swarm + CI/CD + Ansible

A complete DevOps solution for deploying a Yii2 PHP application using Docker Swarm, NGINX reverse proxy, GitHub Actions CI/CD, and Ansible automation.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [CI/CD Pipeline](#cicd-pipeline)
- [Monitoring](#monitoring)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Assumptions](#assumptions)

## 🏗️ Architecture Overview

```
Internet → AWS Application Load Balancer → EC2 Instance
                                            └── NGINX (Host) → Docker Swarm
```

### Components:
- **Yii2 Application**: PHP framework containerized with Docker
- **Docker Swarm**: Container orchestration in single-node mode
- **NGINX**: Host-based reverse proxy (not containerized)
- **GitHub Actions**: CI/CD pipeline for automated deployment
- **Ansible**: Infrastructure automation and configuration management

## 🔧 Prerequisites

### Local Development:
- Docker Desktop installed
- Git
- Text editor (VS Code recommended)

### AWS Setup:
- AWS Account with EC2 access
- Ubuntu 20.04/22.04 EC2 instance (t2.micro or larger)
- Security Group allowing ports: 22 (SSH), 80 (HTTP), 443 (HTTPS)
- Key pair for SSH access

### GitHub Setup:
- GitHub account
- Docker Hub account (or GitHub Container Registry)

## 📁 Project Structure

```
yii2-devops-assessment/
├── .github/
│   └── workflows/
│       └── deploy.yml                 # GitHub Actions CI/CD
├── ansible/
│   ├── inventory/
│   │   └── hosts.yml                  # Server inventory
│   ├── playbooks/
│   │   ├── site.yml                   # Main playbook
│   │   ├── docker-setup.yml           # Docker installation
│   │   └── nginx-setup.yml            # NGINX configuration
│   ├── roles/
│   │   ├── docker/                    # Docker role
│   │   └── nginx/                     # NGINX role
│   └── group_vars/
│       └── all.yml                    # Global variables
├── config/
│   ├── nginx/
│   │   └── yii2-app.conf             # NGINX site configuration
│   └── docker/
│       └── docker-compose.yml        # Docker Swarm stack
├── yii2-app/                         # Yii2 application code
│   ├── web/
│   │   └── index.php
│   ├── config/
│   ├── runtime/
│   └── vendor/
├── Dockerfile                        # Application container
├── docker-compose.yml               # Local development
├── docker-stack.yml                # Production swarm stack
└── README.md                       # This file
```

## 🚀 Setup Instructions

### Step 1: Clone Repository

```bash
git clone https://github.com/your-username/yii2-devops-assessment.git
cd yii2-devops-assessment
```

### Step 2: Configure AWS EC2 Instance

1. Launch Ubuntu 20.04 EC2 instance
2. Configure Security Group:
   ```
   SSH (22)     - Your IP
   HTTP (80)    - 0.0.0.0/0
   HTTPS (443)  - 0.0.0.0/0
   ```
3. Note down the public IP address

### Step 3: Configure Ansible Inventory

Update `ansible/inventory/hosts.yml`:

```yaml
all:
  hosts:
    production:
      ansible_host: YOUR_EC2_PUBLIC_IP
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ~/.ssh/your-key.pem
  vars:
    domain_name: your-domain.com  # Or use IP
    app_name: yii2-app
    docker_image: your-dockerhub-username/yii2-app
```

### Step 4: Configure GitHub Secrets

In your GitHub repository, add these secrets:

```
AWS_EC2_HOST=your-ec2-public-ip
AWS_EC2_USER=ubuntu
AWS_SSH_PRIVATE_KEY=contents-of-your-private-key
DOCKER_USERNAME=your-dockerhub-username
DOCKER_PASSWORD=your-dockerhub-password
```

### Step 5: Run Ansible Playbook

```bash
cd ansible
ansible-playbook -i inventory/hosts.yml playbooks/site.yml
```

This will:
- Install Docker and Docker Compose
- Set up Docker Swarm
- Install and configure NGINX
- Set up Prometheus monitoring
- Deploy the Yii2 application

### Step 6: Deploy Application

Push code to main branch to trigger CI/CD:

```bash
git add .
git commit -m "Initial deployment"
git push origin main
```

## 🔄 CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yml`) performs:

1. **Build Stage**:
   - Checkout code
   - Build Docker image
   - Push to Docker Hub

2. **Deploy Stage**:
   - SSH into EC2 instance
   - Pull latest image
   - Update Docker Swarm service
   - Verify deployment

3. **Rollback** (on failure):
   - Revert to previous image version
   - Restart services

### Manual Deployment Commands

```bash
# SSH into EC2
ssh -i ~/.ssh/your-key.pem ubuntu@your-ec2-ip

# Update application
docker service update --image your-username/yii2-app:latest yii2-stack_yii2-app

# Check service status
docker service ls
docker service ps yii2-stack_yii2-app
```

## 🧪 Testing

### Local Testing

```bash
# Build and run locally
docker build -t yii2-app .
docker run -p 8080:80 yii2-app

# Test with curl
curl http://localhost:8080
```

### Production Testing

```bash
# Test application
curl http://your-domain-or-ip

# Test health endpoint
curl http://your-domain-or-ip/health

# Test SSL (if configured)
curl -k https://your-domain-or-ip

# Check Docker services
ssh ubuntu@your-ec2-ip "docker service ls"

# Check NGINX status
ssh ubuntu@your-ec2-ip "sudo systemctl status nginx"

# Check logs
ssh ubuntu@your-ec2-ip "docker service logs yii2-stack_yii2-app"
```

### Load Testing

```bash
# Install Apache Bench
sudo apt-get install apache2-utils

# Basic load test
ab -n 1000 -c 10 http://your-domain-or-ip/
```

## 🔧 Troubleshooting

### Common Issues

**1. Docker Swarm Service Won't Start**
```bash
# Check service logs
docker service logs yii2-stack_yii2-app

# Check node status
docker node ls

# Recreate service
docker stack rm yii2-stack
docker stack deploy -c docker-stack.yml yii2-stack
```

**2. NGINX 502 Bad Gateway**
```bash
# Check NGINX configuration
sudo nginx -t

# Check upstream containers
docker ps

# Restart NGINX
sudo systemctl restart nginx
```

**3. CI/CD Pipeline Fails**
- Verify GitHub Secrets are set correctly
- Check EC2 instance is accessible
- Ensure Docker Hub credentials are valid

**4. Application Logs and Status**
```bash
# Check service logs
docker service logs yii2-stack_yii2-app

# Check service status
docker service ps yii2-stack_yii2-app
```

### Log Locations

```bash
# Application logs
docker service logs yii2-stack_yii2-app

# NGINX logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# System logs
sudo journalctl -u docker
sudo journalctl -u prometheus
```

## 📝 Assumptions

### Infrastructure:
- Single EC2 instance for demonstration (production would use multiple nodes)
- Ubuntu 20.04/22.04 LTS operating system
- Basic AWS security group configuration
- Domain name pointing to EC2 instance (or IP-based access)

### Application:
- Yii2 Basic Application Template
- PHP 8.1 with Apache
- No external database required for demo
- Static file serving through NGINX

### Security:
- SSH key-based authentication
- Basic firewall rules via Security Groups
- SSL/TLS termination at load balancer level
- Secrets managed via GitHub Secrets

### Monitoring:
- Basic Prometheus setup without persistent storage
- Node Exporter for system metrics
- No alerting configured (would add AlertManager in production)

### CI/CD:
- Automated deployment on push to main branch
- Docker Hub as container registry
- Simple rollback mechanism
- No staging environment (would add in production)

## 🎯 Bonus Features Implemented

✅ **GitHub Secrets** for sensitive data management  
✅ **Health Checks** in Docker configuration  
✅ **Prometheus + Node Exporter** monitoring  
✅ **Rollback mechanism** in CI/CD pipeline  
✅ **Infrastructure as Code** with Ansible  
✅ **Multi-environment support** (dev/prod configurations)  

## 🚀 Next Steps for Production

1. **High Availability**: Multi-node Docker Swarm cluster
2. **Load Balancing**: AWS Application Load Balancer
3. **Database**: RDS MySQL/PostgreSQL
4. **Caching**: Redis/ElastiCache
5. **Storage**: EFS for shared storage
6. **Security**: WAF, VPC, IAM roles
7. **Monitoring**: CloudWatch, ELK stack
8. **Backup**: Automated database and file backups

## 📞 Support

For issues or questions:
1. Check the troubleshooting section
2. Review logs using provided commands
3. Open GitHub issue with detailed error information

---

**Happy Deploying! 🚀**
