# 🚀 Hosting WordPress on AWS EC2 Free Tier (Step-by-Step Guide)

This tutorial walks you through setting up a **WordPress website on AWS EC2** using the **12-month Free Tier**. You’ll get a domain name connected, configure Nginx, MySQL, and PHP (LEMP stack), and secure your site with a free SSL certificate from Let’s Encrypt.

---

## ✅ Prerequisites

* An [AWS Free Tier account](https://aws.amazon.com/free)
* A **domain name** (from Google Domains, Namecheap, GoDaddy, etc.)
* Basic familiarity with Linux commands
* SSH client (Terminal on macOS/Linux or PuTTY/Windows PowerShell)

---

## 1. Launch an EC2 Instance

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).
2. Navigate to **EC2 → Launch Instance**.
3. Choose:

   * **Ubuntu Server 20.04 LTS** (Free Tier eligible)
   * **Instance type**: `t2.micro` (1 vCPU, 1 GB RAM, free tier)
4. Configure storage: set **30 GB General Purpose SSD** (max free tier).
5. Security groups:

   * Allow **SSH (22)** → My IP
   * Allow **HTTP (80)** → Anywhere
   * Allow **HTTPS (443)** → Anywhere
6. Create and download a **key pair** (e.g., `aws-ec2.pem`).
7. Launch the instance and copy its **Public IP Address**.

---

## 2. Point Your Domain to EC2

At your domain registrar (Google Domains, Namecheap, etc.):

* Add **A Records** pointing to your EC2 public IP:

  ```
  @      → <your_EC2_IP>
  www    → <your_EC2_IP>
  ```
* Wait for **DNS propagation** (can take minutes to hours).

You can check propagation using [DNSChecker.org](https://dnschecker.org).

---

## 3. Connect to Your EC2 Instance

```bash
# Move to the directory where your key file is stored
cd ~/Downloads

# Fix key file permissions
chmod 400 aws-ec2.pem

# Connect via SSH
ssh -i aws-ec2.pem ubuntu@<EC2_Public_IP>
```

---

## 4. Update & Upgrade Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 5. Install LEMP Stack (Linux, Nginx, MariaDB, PHP)

```bash
sudo apt install -y nginx mariadb-server php-fpm php-mysql
```

Check Nginx is running:

```bash
systemctl status nginx
```

Visit `http://<EC2_Public_IP>` → You should see the **Nginx Welcome Page**.

---

## 6. Install WordPress

```bash
cd /var/www
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo rm latest.tar.gz
```

Set correct permissions:

```bash
sudo chown -R www-data:www-data wordpress
sudo find wordpress/ -type d -exec chmod 755 {} \;
sudo find wordpress/ -type f -exec chmod 644 {} \;
```

---

## 7. Configure MySQL for WordPress

Secure MySQL:

```bash
sudo mysql_secure_installation
```

Create a database & user:

```sql
sudo mysql -u root -p

CREATE DATABASE wordpress_db DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 8. Configure Nginx for WordPress

Create a site config:

```bash
sudo nano /etc/nginx/sites-available/wordpress.conf
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/wordpress;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable site and test:

```bash
sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 9. Run WordPress Installer

Visit `http://yourdomain.com` → You should see the WordPress setup page.

Enter:

* Database Name: `wordpress_db`
* Username: `wp_user`
* Password: `StrongPassword123!`
* Database Host: `localhost`

Complete installation → set your **site title, admin user, and password**.

---

## 10. Install Required PHP Modules

```bash
sudo apt install -y php-curl php-dom php-bcmath php-imagick php-zip php-gd
sudo systemctl restart php7.4-fpm
```

---

## 11. Secure with SSL (Let’s Encrypt + Certbot)

Install Certbot:

```bash
sudo snap install core && sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Run Certbot:

```bash
sudo certbot --nginx
```

* Enter your email
* Agree to terms
* Choose your domain (`yourdomain.com` + `www.yourdomain.com`)

Certbot auto-configures Nginx for HTTPS and sets up **auto-renewal**.

Check renewal:

```bash
systemctl list-timers | grep certbot
```

---

## 12. Final WordPress Settings

* Log in to **WordPress Dashboard** → `https://yourdomain.com/wp-admin`
* Go to **Settings → General** and ensure both **WordPress Address** and **Site Address** use `https://`.

---

## 🎉 Done!

You now have:

* WordPress running on **AWS EC2 Free Tier**
* Domain connected with **Nginx + PHP + MariaDB**
* **Free SSL** auto-renewed with Let’s Encrypt

---

👉 Next steps:

* Customize your WordPress theme
* Install essential plugins (SEO, caching, backups)
* Harden your server security

---
