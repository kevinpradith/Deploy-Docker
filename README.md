# Complete Git Workflow: Windows to GitHub to Ubuntu Server

## Phase 1: Setup di Windows (Development Environment)

### Step 1: Install dan Konfigurasi Git
```bash
# Download Git dari https://git-scm.com/download/win
# Install dengan setting default, lalu buka Git Bash di VsCode

# Konfigurasi identitas
git config --global user.name "Kevin Pradith"
git config --global user.email "kevinpradithh@gmail.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true
```

### Step 2: Buat SSH Key untuk GitHub
```bash
# Buat SSH key untuk GitHub
ssh-keygen -t rsa -b 4096 -C "kevinpradithh@gmail.com"
# Tekan Enter untuk lokasi default: C:\Users\Kevin\.ssh\id_rsa

# Start SSH agent
eval $(ssh-agent -s)

# Tambahkan SSH key
ssh-add ~/.ssh/id_rsa

# Tampilkan public key
cat ~/.ssh/id_rsa.pub
```

**Copy output public key**, lalu:
1. Buka GitHub → Settings → SSH and GPG keys → New SSH key
2. Paste public key dan save

### Step 3: Buat SSH Key untuk Server Ubuntu
```bash
# Buat SSH key khusus untuk server
ssh-keygen -t rsa -b 4096 -C "deploy@server" -f ~/.ssh/deploy_key

# Tampilkan public key server
cat ~/.ssh/deploy_key.pub
```
**Simpan output ini untuk step server nanti**

### Step 4: Clone Repository dari GitHub
```bash
# Clone repository (otomatis setup remote 'origin')
git clone https://github.com/kevinpradith/Deploy-Docker.git
cd Deploy-Docker

# Cek remote yang ada
git remote -v
```
Output:
```
origin  https://github.com/kevinpradith/Deploy-Docker.git (fetch)
origin  https://github.com/kevinpradith/Deploy-Docker.git (push)
```

### Step 5: Setup Branch Production
```bash
# Buat dan push branch production
git checkout -b production
git push -u origin production

# Kembali ke main
git checkout main
```

---

## Phase 2: Setup di Ubuntu Server

### Step 6: Test dan Setup SSH Connection
```bash
# Test koneksi SSH (dari Windows Git Bash)
ssh kevin@10.160.40.148
exit

# Copy SSH key ke server
ssh-copy-id -i ~/.ssh/deploy_key kevin@10.160.40.148

# Test koneksi dengan deploy key
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148
```

### Step 7: Install Git di Ubuntu Server
```bash
# Login SSH ke server, lalu jalankan:
sudo apt update
sudo apt install git -y

### Step 8: Setup Repository Structure di Server
```bash

mmkdir /var/www

# Buat folder untuk website
cd /var/www/
sudo mkdir Deploy-Docker
sudo chown $USER:$USER Deploy-Docker
cd Deploy-Docker

# Clone repository dari GitHub
git clone https://github.com/kevinpradith/Deploy-Docker.git .

# Set permission untuk web server
sudo chown -R www-data:www-data /var/www/Deploy-Docker
sudo chmod -R 755 /var/www/Deploy-Docker
```

### Step 9: Buat Auto Deploy Script
```bash
# Buat script untuk auto deploy
sudo nano /usr/local/bin/auto-deploy.sh
```

**Isi file dengan script berikut:**
```bash
#!/bin/bash
echo "Auto deploying from GitHub..."

# Pindah ke directory website
cd /var/www/Deploy-Docker

# Pull perubahan terbaru dari production branch
git fetch origin
git checkout production
git pull origin production

# Stop container lama
docker compose down

# Start container baru
docker compose up -d

echo "Deploy selesai! Docker container running"
echo "Website available at: http://10.160.40.148"

# Show running containers
docker ps
```

```bash
# Buat file executable
sudo chmod +x /usr/local/bin/auto-deploy.sh

# Test script
sudo /usr/local/bin/auto-deploy.sh
```

---

## Phase 3: Kembali ke Windows - Setup Complete Workflow

## Phase 3: Kembali ke Windows - Setup Project Files

---

## Phase 4: Setup Project Files

### Step 10: Buat Docker Configuration Files
```bash
# Pastikan di branch main
git checkout main
```

# Buat file docker-compose.yml
```bash
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

# Buat nginx.conf
```bash
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

### Step 1: Initial Commit dan Push
```bash
# Commit semua file
git add .
git commit -m "feat: setup Docker configuration and sample website"

# Push ke GitHub
git push origin main
```

---

## Phase 5: Deploy ke Server

### Step 11: Deploy ke Production
```bash
# Pindah ke branch production
git checkout production

# Merge dari main
git merge main

# Push ke GitHub
git push origin production
```

### Step 12: Deploy ke Server Ubuntu
```bash
# SSH ke server
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148

# Jalankan script auto deploy
sudo /usr/local/bin/auto-deploy.sh

# Keluar dari server
exit
```

### Step 13: Verify Deployment
```bash
# Test dari Windows
curl http://10.160.40.148

# Atau SSH ke server untuk cek detail
ssh -i ~/.ssh/deploy_key kevin@10.160.40.148
docker ps
docker-compose logs web
curl localhost
exit
```
