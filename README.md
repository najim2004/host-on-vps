# 🚀 Server Setup & Deployment Guide

(Backend + Frontend with Nginx, SSL, PM2, PostgreSQL, Redis)

---

## 🔐 Clone Private Repository

```bash
git clone https://token@github.com/username/repository.git .
```

**Example:**

```bash
git clone https://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@github.com/sojebsikder/jewellery-selling-ecommerce-web-app.git .
```

---

## 🔥 Setup User & Firewall

👉 Fresh server হলে এই ধাপগুলো চালাও

```bash
ufw allow OpenSSH
ufw enable
```

---

## 🌐 Setup Nginx

### 🔹 Install Nginx

```bash
sudo apt update
sudo apt install nginx
```

### 🔹 Adjust Firewall

```bash
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw status
```

---

## 📁 Setup Server Blocks

### 🟦 Backend Directory

```bash
sudo mkdir -p /var/www/backend
sudo chown -R $USER:$USER /var/www/backend
sudo chmod -R 755 /var/www/backend
sudo nano /etc/nginx/sites-available/backend
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
```

---

### 🟩 Frontend Directory

```bash
sudo mkdir -p /var/www/frontend
sudo chown -R $USER:$USER /var/www/frontend
sudo chmod -R 755 /var/www/frontend
sudo nano /etc/nginx/sites-available/frontend
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
```

---

### 🔄 Restart Nginx

```bash
sudo systemctl restart nginx
```

---

## ⚙️ Nginx Configuration

### 🔹 Backend Config

```bash
sudo nano /etc/nginx/sites-available/backend
```

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name backend.saythatsh.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

### 🔹 Frontend Config

```bash
sudo nano /etc/nginx/sites-available/frontend
```

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name saythatsh.com www.saythatsh.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
sudo systemctl restart nginx
```

---

## 🔒 Setup SSL (Certbot)

```bash
sudo snap install core
sudo snap refresh core
sudo apt remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

```bash
sudo certbot --nginx -d saythatsh.com -d www.saythatsh.com -d backend.saythatsh.com
sudo ufw allow 'Nginx HTTPS'
```

---

## 🧩 Setup Application Environment

### 🟢 Install Node.js (v20)

```bash
curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

---

### 🧶 Install Yarn

```bash
sudo npm install -g yarn
```

---

### 🚀 Install PM2

```bash
sudo npm install -g pm2
```

---

## 🐘 Setup PostgreSQL

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql.service
sudo -u postgres psql
```

```sql
ALTER USER postgres PASSWORD 'root';
```

---

## 🔴 Setup Redis

```bash
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
redis-cli ping
```

---

## 📦 Backend Setup

```bash
cd /var/www/backend
git clone https://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@github.com/backbencherstudio/web-messaging-backend.git .
```

```bash
yarn install
npx prisma migrate deploy
yarn build
pm2 start dist/src/main.js --name "backend"
```

---

## 🎨 Frontend Setup

```bash
cd /var/www/frontend
git clone https://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@github.com/backbencherstudio/web-messaging-client-frontend.git .
```

```bash
npm install
pm2 start npm --name "frontend" -- start
```

---

## ✅ Final Notes

* Backend → **Port 4000**
* Frontend → **Port 3000**
* Nginx → Reverse proxy
* PM2 → App process manager
* SSL → Auto-renew via Certbot


