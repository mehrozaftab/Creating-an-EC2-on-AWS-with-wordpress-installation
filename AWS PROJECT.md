### My introductory AWS project to creating a Virtual Private Cloud (VPC)

# Introduction into VPC, subnet and regions:

A Virtual Private Cloud (VPC) is an isolated cloud system within a public cloud environment on which users of that VPC can run their cloud resources efficiently. There are various public cloud services such as Amazon Web Services(AWS), Azure provider and Google Cloud Platform that allow users to customise their own VPC. In this project, our focus was on creating a VPC using the Amazon Web Services. A VPC allows the users to create a specific cloud environment that can be customised to the needs of your projects. Resources such as virtual computing servers, databases and storage can be utilised as to meet the demands of your project. i.e. launching an EC2 instance as a virtual server to host a websites. To isolate the resources in the same VPC, you can use subnets to create a network segmenatation. 

Subnets are various IP addresses in a VPC which aid isolation of resources into segments allowing users to organise their private cloud. A subnet relates to an individual floor in a building in which the VPC represents the entire building. The subnets allow to manage different resources on different floors as for example the 1st floor could host the web servers whereas the 2nd floor could hold the storage whilst being in the same building.

Every VPC created is specific to a region, in which the VPC is bound to that geographic region within the AWS infrastructure.i.e. The eu-west-2 representing the Europe (London) region. To create a VPC, the chosen geographic region would contain multiple data centres known as Availbility Zones (AZs) and to optimise availability, two AZs should be assigned to a VPC.

So without further ado, we shall now learn how to create a VPC.

## Step 1 - Creating a VPC:

After logging into your AWS account and launching your console, search for VPC in the provdided searchbar.

![VPC-SEARCH](VPC-search.png)

Once opened, by clicking create VPC, you will be able to see the settings required to create your own VPC. The name of my VPC is 'project-vpc'. As mentioned before, the subnets are a branch of a VPC. Public subnets can be used for resources that you want to be accessible from the internet therefore public IP for these resources will allow direct communication. IPv4 CIDR block represents a range of IP addresses and a range is given for the block size. IPv6 CIDR block is not required. Tenancy was kept to default rather than dedicated with 2 public and 2 private subnets. Two AZs were assigned for higher availbility. NAT gateways is a security feature allowing private subnets to reach the internet meanwhile ensuring external services cannot connect to private subnet services. A VPC endpoint provides security as it ensures you to connect privately to supported AWS services such as Simple Storage Service (S3) without the need to use a public IP address.

![VPC SETTING ](<VPC settings-1.png>)

## Step 2 - Launching an EC2 instance:

By searching for EC2 and pressing launch instance, we can configure the setting of the virtual computer which will be hosting our wordpress page. Ubuntu was the chosen operating system to launch the instance.
![image](https://github.com/user-attachments/assets/d9b08ae4-7070-4f6a-ab24-96e08e40c84b)

An instance type allows you to customise the computing, memory, networking or storage needs. t2. micro was the designated instance with 1 CPU AND 1 GIB memory that refers to 1GB RAM allowing the virtual compute to store and access upto 1GB data in size at a time.

![instance type](<instance type-1.png>)
In order to connect to your EC2 instance locally, a keypair should be created as an RSA encrypted.

![key](keypair.png)

SSH traffic was enabled to locally connect to the instance whilst HTTPS and HTTP traffic enabling allows an endpoint to set up from the internet which would allow to create a webserver to host wordpress. Enabling pulic IP is vital to ensure the webserver can be publically viewed and ensure that your created VPC public subnet is where this EC2 instance will reside. Now the instance can be launched.
![network](<ec2 network setting.png>)

Instance state is present running which confirms that the EC2 instance has been launched and by pressing the instance ID, the properties of the instance can be accessed.

![ status](<instance status.png>)

## Step 3 - Connecting EC2 instance locally using a Secure Shell (SSH)

Connect to your instance then press SSH client which will give you a chmod 400 command with your key file. Before copying this command to your terminal, ensure you locate your private key file. As the .PEM key will be located in Downloads, change directory to Downloads and list the content of Downloads.
This can be done by using the following commands:

- cd Downloads
- ls
- chmod 400 "your-key.pem"
- ssh -i "your-key.pem" yourusername@yourpublicIP.yourVPCregion.compute.amazonaws.com'

 
Replace "your-key" with the name of your own created key pair
 The chmod 400 will allow you appropriate permissions.
 By using the ssh command from your instance you can now fully connect to your instance on your terminal. The ssh command will appear as 

You will now see that the username on your terminal will now change from your computer device to that of your EC2 instance username
This confirms that now you are connected to your EC2 instance locally.

# Update and upgrade system packages
- sudo apt update -y
- sudo apt upgrade -y


## Step 4 - Installing Apache to be used as web server for wordpress:
- sudo apt install apache2 -y
- sudo systemctl start apache2
- sudo systemctl enable apache2


## Step 5A - Installing MYSQL as the database to store and organise the data for your wordpress page:
# Install MySQL server
- sudo apt install mysql-server -y
- sudo systemctl start mysql
- sudo systemctl enable mysql

# Step 5B - Secure MySQL installation
- sudo mysql_secure_installation <<EOF
- y
- YourPassword
- YourPassword
- y
- y
- EOF

## Step 6 - Install PHP and required PHP extensions:

- sudo apt install php libapache2-mod-php php-mysql php-curl php-json php-cgi -y

## Step 7A - Create WordPress database and user:

- sudo mysql -u root -p'YourPassword.' <<EOF
- CREATE DATABASE wordpress;
- CREATE USER 'YourUser'@'localhost' IDENTIFIED BY 'YourPassword.';
- GRANT ALL PRIVILEGES ON wordpress.* TO 'YourUser'@'localhost';
- FLUSH PRIVILEGES;
- EOF


## Step 7B - Download and extract WordPress:

- cd /var/www/html
- sudo wget https://wordpress.org/latest.tar.gz
- sudo tar -xvzf latest.tar.gz
- sudo rm latest.tar.gz

## Step 8 - ensuring that your wordpress file is entailed into your web directory:

# Move WordPress files
- sudo rsync -av wordpress/* /var/www/html/
- sudo rm -rf wordpress

the rsync command copies all files from the extracted wordpress directory to the /var/www/html/ directory.
The rm -rf wordpress command removes the now-empty wordpress directory to clean up after the transfer.

## Step 9 - Set permissions:

- sudo chown -R www-data:www-data /var/www/html/
- sudo chmod -R 755 /var/www/html/

## Step 10 - Configure wp-config.php

- sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
- sudo sed -i "s/database_name_here/wordpress/" /var/www/html/wp-config.php
- sudo sed -i "s/username_here/YourUser/" /var/www/html/wp-config.php
- sudo sed -i "s/password_here/YourPassword./" /var/www/html/wp-config.php
- sudo sed -i "s/localhost/localhost/" /var/www/html/wp-config.php

## Step 11 - Configure Apache virtual host:

![apache config](<Apache virtual host config.png>)

Ensure the configuration file for the default virtual host is exactly the same as above.

## Step 12 - Enable Apache mod_rewrite

- sudo a2enmod rewrite

## Step 13 - Restart Apache to apply changes
- sudo systemctl restart apache2

## Step 14 - Backup default index.html if present
- if [ -f /var/www/html/index.html ]; then
    mv /var/www/html/index.html /var/www/html/index.html.bak
- fi


## Step 15 - Launching Wordpress page:

Now by going back into your EC2 instance, connect to the instance and by copying your public IP address into the url as http://yourpublicIPaddress, the wordpress page will open with language options that you can proceed with and create your wordpress page
 ![page](mywordpress.png)


### Additional Learning Step (OPTIONAL):

In order to deploy the wordpress page onto your EC2 instance without using an SSH and connecting to your local device shell, you can enter the steps taken to deploy wordpress into a bash script. This bash script can then by entered on the user data on the advanced settings when launching your EC2 instance as seen on image below.

![Advanced](<advance settinngs.png>)

Alternatively, if an EC2 instance is launched, you can stop the instance, then edit user data, enter your batch script and re-launch the instance following rebooting. This allows the automation of your wordpress deployment without having to locally connect to your instance. The bash script for this project will be found in the README.md file.

