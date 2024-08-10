# AWS Project Brief:

To create a virtual private cloud (VPC) and launch an EC2 instance within a subnet of the VPC in which wordpress will be installed then deployed onto the EC2 instance.

# Aim:

In this project I aim to learn how a VPC works and how a virtual computer can be launched to run a wordpress page. This project will help myself gain an introductory insight on how to run resources such as an EC2 on my virtual private cloud and how to locally connect my cloud computer to my local device in order to deploy a web page on the server

# Bash script to automise your EC2 instance with deployment of wordpress page:

#!/bin/bash

# Update and upgrade system packages
- apt update -y
- apt upgrade -y

# Install Apache
- apt install apache2 -y
- systemctl start apache2
- systemctl enable apache2

# Install MySQL
{- apt install mysql-server -y
- systemctl start mysql
- systemctl enable mysql}

# Secure MySQL installation
- mysql_secure_installation <<EOF

- y
- YourPassword
-YourPassword
- y
- y
-EOF

# Install PHP and required PHP extensions
- apt install php libapache2-mod-php php-mysql php-curl php-json php-cgi -y

# Create WordPress database and user
- mysql -u root -p'YourPassword' <<EOF
- CREATE DATABASE wordpress;
- CREATE USER 'YourUser'@'localhost' IDENTIFIED BY 'YourPassword';
- GRANT ALL PRIVILEGES ON wordpress.* TO 'YourUser'@'localhost';
- FLUSH PRIVILEGES;
- EOF

# Download and extract WordPress
- cd /var/www/html
- wget https://wordpress.org/latest.tar.gz
- tar -xvzf latest.tar.gz
- rm latest.tar.gz

# Move WordPress files
- rsync -av wordpress/* /var/www/html/
- rm -rf wordpress

# Set permissions
- chown -R www-data:www-data /var/www/html/
- chmod -R 755 /var/www/html/

# Configure wp-config.php
- cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
- sed -i "s/database_name_here/wordpress/" /var/www/html/wp-config.php
- sed -i "s/username_here/YourUser/" /var/www/html/wp-config.php
- sed -i "s/password_here/YourPassword/" /var/www/html/wp-config.php
- sed -i "s/localhost/localhost/" /var/www/html/wp-config.php

# Configure Apache virtual host
- echo "<VirtualHost *:80>
    - DocumentRoot /var/www/html
    - <Directory /var/www/html>
        - Options Indexes FollowSymLinks
        - AllowOverride All
        - Require all granted
    - </Directory>
- </VirtualHost>" | tee /etc/apache2/sites-available/000-default.conf

# Enable Apache mod_rewrite
- a2enmod rewrite

# Restart Apache to apply changes
- systemctl restart apache2

# Backup default index.html if present
- if [ -f /var/www/html/index.html ]; then
    - mv /var/www/html/index.html /var/www/html/index.html.bak
- fi

- echo "WordPress setup complete! Access your site using the server's public IP address."
