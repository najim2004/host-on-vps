# 🚀 Production Server Setup & Deployment Guide

**(Backend + Frontend with Nginx, SSL, PM2, PostgreSQL, Redis)**

> ✅ Ubuntu 20.04 / 22.04
> ✅ Node.js 20
> ✅ Secure + Production Ready

---

## 🧱 1. Initial Server Preparation

### 🔹 Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### 🔹 Set Timezone (Optional but Recommended)

```bash
sudo timedatectl set-timezone Asia/Dhaka
```

---

## 👤 2. Create Non-Root User (Recommended)

```bash
adduser deploy
usermod -aG sudo deploy
su - deploy
```

---

## 🔐 3. Firewall (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
sudo ufw status
```

---

## 🔑 4. Git & SSH Setup (Private Repo – Secure Way)

### 🔹 Install Git

```bash
sudo apt install git -y
```

### 🔹 Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "server"
cat ~/.ssh/id_ed25519.pub
```

👉 **GitHub → Settings → SSH Keys → Add**

```bash
ssh -T git@github.com
```

✅ **Avoid using GitHub tokens inside commands (security risk)**

---

## 🌐 5. Install & Setup Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

```bash
sudo ufw allow 'Nginx Full'
```

---

## 📁 6. Project Directory Structure

```bash
sudo mkdir -p /var/www/backend /var/www/frontend
sudo chown -R deploy:deploy /var/www
sudo chmod -R 755 /var/www
```

---

## ⚙️ 7. Nginx Server Blocks

### 🔹 Backend Config

```bash
sudo nano /etc/nginx/sites-available/backend
```

```nginx
server {
    listen 80;
    server_name backend.saythatsh.com;

    location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
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
    server_name saythatsh.com www.saythatsh.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 8. SSL Setup (Let’s Encrypt)

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

```bash
sudo certbot --nginx \
-d saythatsh.com \
-d www.saythatsh.com \
-d backend.saythatsh.com
```

🔁 Auto-renew test:

```bash
sudo certbot renew --dry-run
```

---

## 🟢 9. Install Node.js (v20)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y
node -v
```

---

## 🧶 10. Install Yarn & PM2

```bash
sudo npm install -g yarn pm2
pm2 startup
```

👉 copy & run generated command

---

## 🐘 11. PostgreSQL Setup (Secure)

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE app_db;
CREATE USER app_user WITH PASSWORD 'strong_password';
ALTER ROLE app_user SET client_encoding TO 'utf8';
ALTER ROLE app_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE app_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE app_db TO app_user;
\q
```

❌ **Never use postgres/root password in production**

---

## 🔴 12. Redis Setup

```bash
sudo apt install redis-server -y
sudo systemctl enable redis-server
sudo systemctl start redis-server
redis-cli ping
```

🔐 Optional:

```bash
sudo nano /etc/redis/redis.conf
# set requirepass yourpassword
```

---

## 📦 13. Backend Deployment

```bash
cd /var/www/backend
git clone git@github.com:backbencherstudio/web-messaging-backend.git .
```

### 🔹 Environment Variables

```bash
nano .env
```

```env
NODE_ENV=production
PORT=4000
DATABASE_URL=postgresql://app_user:password@localhost:5432/app_db
REDIS_URL=redis://127.0.0.1:6379
JWT_SECRET=supersecret
```

### 🔹 Build & Run

```bash
yarn install
npx prisma migrate deploy
yarn build
pm2 start dist/src/main.js --name backend
```

---

## 🎨 14. Frontend Deployment

```bash
cd /var/www/frontend
git clone git@github.com:backbencherstudio/web-messaging-client-frontend.git .
```

```bash
nano .env
```

```env
NEXT_PUBLIC_API_URL=https://backend.saythatsh.com
```

```bash
npm install
npm run build
pm2 start npm --name frontend -- start
```

---

## ♻️ 15. PM2 Management

```bash
pm2 list
pm2 logs
pm2 restart backend
pm2 save
```

---

## 📊 16. Monitoring & Logs (Optional)

```bash
pm2 monit
sudo tail -f /var/log/nginx/error.log
```

---

## 🔐 17. Security Hardening (Recommended)

* ❌ Disable root SSH login
* 🔑 Use SSH key only
* 🔄 Enable unattended upgrades

```bash
sudo apt install unattended-upgrades
```

---

## ✅ Final Architecture

| Service    | Port   |
| ---------- | ------ |
| Frontend   | 3000   |
| Backend    | 4000   |
| PostgreSQL | 5432   |
| Redis      | 6379   |
| Nginx      | 80/443 |

---

## 🏁 Done 🎉

**Production-grade, secure, scalable setup ready 🚀**
