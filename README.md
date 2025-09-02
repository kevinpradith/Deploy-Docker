# Complete Git Workflow: Windows to GitHub to Ubuntu Server

A comprehensive guide for setting up a complete development and deployment workflow from Windows to GitHub to Ubuntu Server using Docker.

## Table of Contents
- [Phase 1: Windows Development Environment Setup](#phase-1-windows-development-environment-setup)
- [Phase 2: Ubuntu Server Setup](#phase-2-ubuntu-server-setup)
- [Phase 3: Return to Windows - Complete Workflow](#phase-3-return-to-windows---complete-workflow)
- [Phase 4: Project Files Setup](#phase-4-project-files-setup)
- [Phase 5: Server Deployment](#phase-5-server-deployment)

---

## Phase 1: Windows Development Environment Setup

### Step 1: Install and Configure Git

Download Git from [https://git-scm.com/download/win](https://git-scm.com/download/win), install with default settings, then open Git Bash in VSCode.

```bash
# Configure identity
git config --global user.name "Kevin Pradith"
git config --global user.email "kevinpradithh@gmail.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true
```

### Step 2: Create SSH Key for GitHub

```bash
# Create SSH key for GitHub
ssh-keygen -t rsa -b 4096 -C "kevinpradithh@gmail.com"
# Press Enter for default location: C:\Users\Kevin\.ssh\id_rsa

# Start SSH agent
eval $(ssh-agent -s)

# Add SSH key
ssh-add ~/.ssh/id_rsa

# Display public key
cat ~/.ssh/id_rsa.pub
```

**Copy the public key output**, then:
1. Open GitHub → Settings → SSH and GPG keys → New SSH key
2. Paste public key and save

### Step 3: Create SSH Key for Ubuntu Server

```bash
# Create dedicated SSH key for server
ssh-keygen -t rsa -b 4096 -C "deploy@server" -f ~/.ssh/deploy_key

# Display server public key
cat ~/.ssh/deploy_key.pub
```
**Save this output for the server setup step**

### Step 4: Clone Repository from GitHub

```bash
# Clone repository (automatically sets up 'origin' remote)
git clone https://github.com/kevinpradith/Deploy-Docker.git
cd Deploy-Docker

# Check existing remotes
git remote -v
```

Expected output:
```
origin  https://github.com/kevinpradith/Deploy-Docker.git (fetch)
origin  https://github.com/kevinpradith/Deploy-Docker.git (push)
```

### Step 5: Setup Production Branch

```bash
# Create and push production branch
git checkout -b production
git push -u origin production

# Return to main branch
git checkout main
```

---

## Phase 2: Ubuntu Server Setup

### Step 6: Test and Setup SSH Connection

```bash
# Test SSH connection (from Windows Git Bash)
ssh kevin@10.160.40.148
exit

# Copy SSH key to server
ssh-copy-id -i ~/.ssh/deploy_key kevin@10.160.40.148

# Test connection with deploy key
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148
```

### Step 7: Install Git on Ubuntu Server

Login via SSH to server, then run:

```bash
sudo apt update
sudo apt install git -y
```

### Step 8: Setup Repository Structure on Server

```bash
mkdir /var/www

# Create folder for website
cd /var/www/
sudo mkdir Deploy-Docker
sudo chown $USER:$USER Deploy-Docker
cd Deploy-Docker

# Clone repository from GitHub
git clone https://github.com/kevinpradith/Deploy-Docker.git .

# Set permissions for web server
sudo chown -R www-data:www-data /var/www/Deploy-Docker
sudo chmod -R 755 /var/www/Deploy-Docker
```

### Step 9: Create Auto Deploy Script

```bash
# Create script for auto deployment
sudo nano /usr/local/bin/auto-deploy.sh
```

**File content:**

```bash
#!/bin/bash
echo "Auto deploying from GitHub..."

# Navigate to website directory
cd /var/www/Deploy-Docker

# Pull latest changes from production branch
git fetch origin
git checkout production
git pull origin production

# Stop old containers
docker compose down

# Start new containers
docker compose up -d

echo "Deploy complete! Docker container running"
echo "Website available at: http://10.160.40.148"

# Show running containers
docker ps
```

Make the script executable and test:

```bash
# Make file executable
sudo chmod +x /usr/local/bin/auto-deploy.sh

# Test script
sudo /usr/local/bin/auto-deploy.sh
```

---

## Phase 3: Return to Windows - Complete Workflow

## Phase 4: Project Files Setup

### Step 10: Create Docker Configuration Files

```bash
# Ensure on main branch
git checkout main
```

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - .:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    container_name: Deploy-Docker-web
    restart: unless-stopped
```

Create `nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

### Step 11: Initial Commit and Push

```bash
# Commit all files
git add .
git commit -m "feat: setup Docker configuration and sample website"

# Push to GitHub
git push origin main
```

---

## Phase 5: Server Deployment

### Step 12: Deploy to Production

```bash
# Switch to production branch
git checkout production

# Merge from main
git merge main

# Push to GitHub
git push origin production
```

### Step 13: Deploy to Ubuntu Server

```bash
# SSH to server
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148

# Change ownership
sudo chown -R kevin:kevin /var/www/Deploy-Docker

# Run auto deploy script
sudo /usr/local/bin/auto-deploy.sh

# Exit from server
exit
```

### Step 14: Verify Deployment

```bash
# Test from Windows
curl http://10.160.40.148

# Or SSH to server for detailed check
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148
docker ps
docker-compose logs web
curl localhost
exit
```

---

## 🚀 Quick Commands Summary

### Development Workflow (Windows)
```bash
# Make changes to code
git add .
git commit -m "your commit message"
git push origin main

# Deploy to production
git checkout production
git merge main
git push origin production
```

### Server Deployment (Ubuntu)
```bash
# SSH to server and run auto deploy
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148
sudo /usr/local/bin/auto-deploy.sh
```

### Verification
```bash
# Check website
curl http://10.160.40.148

# Check Docker containers
docker ps
```

---

## 📋 Prerequisites

- **Windows**: Git, VSCode, SSH client
- **Ubuntu Server**: Docker, Docker Compose, Git
- **GitHub Account**: With SSH key configured
- **Network Access**: Windows machine can reach Ubuntu server

---

## 🔧 Troubleshooting

### Common Issues

1. **SSH Permission Denied**: Ensure SSH keys are properly configured
2. **Docker Permission Issues**: Check file ownership and permissions
3. **Port 80 Blocked**: Verify firewall settings on Ubuntu server
4. **Git Merge Conflicts**: Resolve conflicts before merging branches

### Useful Commands

```bash
# Check SSH connection
ssh -T git@github.com

# Check Docker status
docker ps -a
docker-compose logs

# Check Git status
git status
git branch -a
```
