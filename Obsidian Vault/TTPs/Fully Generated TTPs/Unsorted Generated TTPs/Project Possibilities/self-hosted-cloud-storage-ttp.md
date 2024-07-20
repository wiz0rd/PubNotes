# Self-Hosted Cloud Storage System TTP

## Project Goal
The goal of this project is to set up a self-hosted cloud storage solution. By the end of this project, you will have your own private, secure, and customizable cloud storage system that provides file synchronization, sharing capabilities, and access control.

## Overview
This TTP provides detailed instructions for setting up a self-hosted cloud storage system using Nextcloud, an open-source file hosting solution.

![Nextcloud Overview](https://example.com/path/to/nextcloud_overview.png)
*Figure 1: Nextcloud System Overview. Replace with an actual diagram showing the components of your Nextcloud setup.*

## Prerequisites
- A server or VPS with at least 2GB RAM and 20GB storage
- Ubuntu 20.04 LTS or later installed
- Domain name pointed to your server's IP address
- Basic understanding of Linux system administration
- Familiarity with web servers and databases

## Step-by-step Guide

### 1. Update System and Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 mariadb-server libapache2-mod-php7.4 \
php7.4-gd php7.4-mysql php7.4-curl php7.4-mbstring php7.4-intl \
php7.4-gmp php7.4-bcmath php-imagick php7.4-xml php7.4-zip -y
```

### 2. Configure MariaDB

```bash
sudo mysql_secure_installation
```

Create a database for Nextcloud:

```sql
sudo mysql -u root -p
CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Download and Install Nextcloud

```bash
cd /tmp
wget https://download.nextcloud.com/server/releases/latest.tar.bz2
sudo tar -xjf latest.tar.bz2 -C /var/www/html/
sudo chown -R www-data:www-data /var/www/html/nextcloud
```

### 4. Configure Apache

Create a new Apache configuration file:

```bash
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

Add the following content:

```apache
<VirtualHost *:80>
    DocumentRoot "/var/www/html/nextcloud"
    ServerName your_domain.com

    <Directory "/var/www/html/nextcloud/">
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
        SetEnv HOME /var/www/html/nextcloud
        SetEnv HTTP_HOME /var/www/html/nextcloud
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the new configuration and necessary modules:

```bash
sudo a2ensite nextcloud.conf
sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2
```

### 5. Set Up SSL/TLS with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d your_domain.com
```

### 6. Complete Nextcloud Setup

Navigate to `https://your_domain.com` in a web browser and complete the setup wizard.

![Nextcloud Setup](https://example.com/path/to/nextcloud_setup.png)
*Figure 2: Nextcloud Setup Wizard. Replace with an actual screenshot of the Nextcloud setup process.*

### 7. Configure Nextcloud

Edit the Nextcloud configuration file:

```bash
sudo nano /var/www/html/nextcloud/config/config.php
```

Add or modify the following lines:

```php
'trusted_domains' => 
  array (
    0 => 'your_domain.com',
  ),
'default_phone_region' => 'YOUR_COUNTRY_CODE',
```

### 8. Set Up Automated Updates

Create a new script:

```bash
sudo nano /usr/local/bin/nextcloud_update.sh
```

Add the following content:

```bash
#!/bin/bash
sudo -u www-data php /var/www/html/nextcloud/updater/updater.phar
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/nextcloud_update.sh
```

Add a cron job:

```bash
sudo crontab -e
```

Add this line:

```
0 1 * * 6 /usr/local/bin/nextcloud_update.sh
```

### 9. Enable and Configure Nextcloud Apps

Log in to your Nextcloud instance as an admin and navigate to the Apps section. Enable and configure apps like:
- Calendar
- Contacts
- Talk (for communication)
- Notes

### 10. Set Up Client Applications

Download and install Nextcloud clients on your devices:
- [Desktop Clients](https://nextcloud.com/install/#install-clients)
- [Mobile Apps](https://nextcloud.com/install/#install-clients)

Configure the clients to connect to your Nextcloud instance.

![Nextcloud Clients](https://example.com/path/to/nextcloud_clients.png)
*Figure 3: Nextcloud Client Applications. Replace with actual screenshots of Nextcloud client apps on various devices.*

## Advanced Topics

- Implementing server-side encryption
- Setting up external storage support
- Configuring LDAP user authentication
- Implementing two-factor authentication
- Setting up a reverse proxy with Nginx

## Troubleshooting

- If you encounter permission issues, check file ownership and permissions
- For database connection errors, verify database credentials in config.php
- If uploads fail, check PHP configuration for file size limits
- Use the Nextcloud log file (/var/www/html/nextcloud/data/nextcloud.log) for debugging

## Best Practices

1. Regularly backup your Nextcloud instance and database
2. Keep your Nextcloud installation and all plugins up to date
3. Use strong, unique passwords for all accounts
4. Implement two-factor authentication for added security
5. Regularly review and update access controls and sharing policies
6. Monitor server resources and scale as necessary

## Additional Resources

- [Nextcloud Admin Manual](https://docs.nextcloud.com/server/latest/admin_manual/)
- [Nextcloud User Manual](https://docs.nextcloud.com/server/latest/user_manual/)
- [Nextcloud Security Guide](https://docs.nextcloud.com/server/latest/admin_manual/installation/harden_server.html)
- [Nextcloud Community Forums](https://help.nextcloud.com/)

## Glossary

- **Nextcloud**: An open-source, self-hosted file share and collaboration platform
- **Apache**: A widely-used web server software
- **MariaDB**: A community-developed fork of the MySQL relational database management system
- **Let's Encrypt**: A free, automated, and open certificate authority providing SSL/TLS certificates
- **PHP**: A popular general-purpose scripting language suited for web development
- **Cron Job**: A time-based job scheduler in Unix-like operating systems
- **LDAP**: Lightweight Directory Access Protocol, used for accessing and maintaining distributed directory information services
- **Two-Factor Authentication (2FA)**: An extra layer of security used to ensure that people trying to access an online account are who they say they are

