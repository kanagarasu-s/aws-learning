
# Setup Certbot SSL on Apache with Domain

## 1. Prerequisites

* A server running **Ubuntu/Debian** or **RHEL/CentOS/AlmaLinux**.
* Apache web server installed and running.
* A valid **domain name** pointing to the server’s **public IP address** (configured in DNS A record).
* `sudo` or root access to the server.

## 2. Install Apache

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install apache2 -y

# RHEL / CentOS / AlmaLinux
sudo yum install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
```

## 3. Install Certbot

### Ubuntu / Debian

```bash
sudo apt install certbot python3-certbot-apache -y
```

### RHEL / CentOS / AlmaLinux

```bash
sudo yum install epel-release -y
sudo yum install certbot python3-certbot-apache -y
```

## 4. Configure Apache Virtual Host for Domain

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/html/example

    <Directory /var/www/html/example>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example_error.log
    CustomLog ${APACHE_LOG_DIR}/example_access.log combined
</VirtualHost>
```

Enable site and reload Apache:

```bash
sudo a2ensite example.com.conf
sudo systemctl reload apache2
```

## 5. Obtain SSL Certificate with Certbot

```bash
sudo certbot --apache -d example.com -d www.example.com
```

* Certbot will automatically configure SSL in Apache.
* Choose whether to **redirect HTTP → HTTPS**.

## 6. Verify SSL

* Open browser: `https://example.com`
* Or test:

```bash
curl -I https://example.com
```

## 7. Auto Renewal

Certbot automatically adds a cron job or systemd timer.
Test renewal with:

```bash
sudo certbot renew --dry-run
```

## 8. SSL File Location (for reference)

* Certificates: `/etc/letsencrypt/live/example.com/fullchain.pem`
* Private Key: `/etc/letsencrypt/live/example.com/privkey.pem`
