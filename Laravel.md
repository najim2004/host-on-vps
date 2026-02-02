# Universal Laravel Production Deployment Guide

**Target OS:** Ubuntu 22.04 / 24.04 LTS

**Laravel Versions:** 10 / 11 / 12

**Web Stack:** Nginx, PHP-FPM, MySQL, Redis (optional), Supervisor (optional), SSL

This guide is a **professional, reusable production deployment reference** for any Laravel application. Each step includes a short explanation of **why it’s done**, and optional components indicate when they can be skipped.

---

## Step 1: Initial backend Preparation

**Purpose:** Ensure the backend is updated and stable.

```bash
sudo apt update && sudo apt upgrade -y
sudo timedatectl set-timezone UTC

```

Skip if already updated.

---

## Step 2: Create a Dedicated Deployment User

**Purpose:** Avoid running apps as root; isolates the app.

```bash
adduser deployer
usermod -aG sudo deployer
su - deployer

```

Skip if a non-root user already exists.

---

## Step 3: Firewall Configuration (UFW)

**Purpose:** Restrict incoming traffic to necessary services.

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable

```

Skip if managed externally.

---

## Step 4: Install PHP & Core Packages

**Purpose:** Install PHP and common Laravel dependencies.

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install -y php8.4-fpm php8.4-mysql php8.4-mbstring php8.4-xml \
php8.4-bcmath php8.4-curl php8.4-zip php8.4-gd php8.4-redis php8.4-intl \
nginx mysql-backend git unzip curl

```

Skip Redis if unused.

---

## Step 5: MySQL Database Setup

**Purpose:** Create an isolated database and user.

```bash
sudo mysql_secure_installation
sudo mysql -u root -p

```

```sql
CREATE DATABASE app_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON app_database.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```

Skip if using managed DB.

---

## Step 6: Project Deployment & Permissions

**Purpose:** Correct placement and permissions.

```bash
sudo mkdir -p /var/www/backend
sudo chown -R deployer:www-data /var/www/backend
cd /var/www/backend
git clone git@github.com:username/repo.git .
composer install --no-dev --optimize-autoloader

```

```bash
sudo find /var/www/backend -type d -exec chmod 775 {} \;
sudo find /var/www/backend -type f -exec chmod 664 {} \;
sudo chown -R deployer:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

```

---

## Step 7: Nginx backend Configuration

**Purpose:** Serve Laravel securely and efficiently.

```bash
sudo nano /etc/nginx/sites-available/backend.conf

```

```
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/backend/public;
    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\. {
        deny all;
    }
}

```

```bash
sudo ln -s /etc/nginx/sites-available/backend.conf /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx

```

---

## Step 8: SSL Configuration (HTTPS)

**Purpose:** Secure traffic, required for payment gateways.

```bash
sudo apt install python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com

```

Skip if behind SSL proxy.

---

## Step 9: Queue Worker Management (Supervisor)

**Purpose:** Keep background jobs running reliably.

```bash
sudo apt install supervisor -y
sudo nano /etc/supervisor/conf.d/backend-worker.conf

```

```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/backend/artisan queue:work --sleep=3 --tries=3
autostart=true
autorestart=true
user=deployer
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/backend/storage/logs/worker.log

```

```bash
sudo supervisorctl reread && sudo supervisorctl update && sudo supervisorctl start all

```

Skip if no queues.

---

## Step 10: Application Initialization

**Purpose:** Prepare Laravel for production.

```bash
cp .env.example .env
nano .env
php artisan key:generate
php artisan migrate --force
php artisan optimize

```

Ensure `APP_ENV=production` and `APP_DEBUG=false`.

---

## Step 11: Deployment Update Script (Optional)

**Purpose:** Standardized, repeatable deployments.

```bash
#!/bin/bash

git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan optimize
sudo systemctl restart php8.4-fpm

```

---

## Step 12: Optional Enhancements for Professional Production

**Purpose:** Improve performance, security, and reliability. Each step shows **how to implement it**, with notes on skipping if not needed.

- **Laravel Cache & Optimization:**

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache

```

Skip if caching not required.

- **Queue Restart:**

```bash
php artisan queue:restart

```

Skip if queues not used.

---

## Step 13: Common Production Issues

- 500 errors: Check `.env` and permissions.
- Cached config: Run `php artisan optimize:clear`.
- Payment failures: Ensure HTTPS.
- PHP version mismatch: Verify with `composer.json`.
- Default Nginx page: Remove `/etc/nginx/sites-enabled/default`.

---

## Final Notes

- Never commit `.env` files.
- Test migrations before production.
- Monitor logs in `storage/logs`.
- Keep PHP, Laravel, and system packages updated.

**All optional steps in Step 12 now include implementation instructions and skip notes.**
