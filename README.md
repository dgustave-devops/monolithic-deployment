# monolithic-deployment
This repository contains a guide for a distributed system deployment of the open source Moodle application on Amazon Web Services (AWS).

### AWS Components:
- Virtual Private Cloud (VPC)
- Subnet
- Internet Gateway
- Security Groups
- Availability Zones (AZs)

### Launching an EC2 Instance
1. Open up your AWS Console and select the EC2 Service.\
2. Select "Instances" from the drop down on the left.\
3. Select "Launch instances" in the top right orange button.\
4. Provide an appropriate name for the instance in the name field, under the "Name and tags" section.\
5. Under the "OS and images" section, we are going to select a linux Ubuntu image (24.04 LTS).\
6. Under the "Instance type" section, we are going to select a "t2.micro" instancee for demonstration purposes. Details on the types of AWS instances are available here.
https://aws.amazon.com/ec2/instance-types/\

7. Under the "Key pair" section, select create a new key pair. Select "RSA" for type and ".pem" for format. Provide an appropriate name, and click create key pair. Your private keypair will automatically be downloaded to your machine.\
8. Under the "Network settings" section, Click "edit" on the corner to the right of the section heading, and enable auto-assign public IP. This will provide the EC2 instance with a publicly accessible IP address (Please not this is not a static Ip and may change when the instance is shutdown and start, or restarted. For a static IP, please look into aws elastic IPs. we'll use the default VPC and subnet for demonstration purposes, and create a new security group that allows inbound traffic on ports 80, 443, and 22.\
9. Under the "Configure storage" section, configure an 8GB GP3 EBS volume.\
10. Scroll down to the bottom of the page and select launch instance.\

### Preparing EC2 instance

Please note:
AWS no longer offers free public IP addresses and instance assigned a public IP will be charged accordingly. In adddition to this change in policy, newly launched instance will no longer be automatically assigned a public IP address. Therefore after launching an EC2 instance, to remotely access through a machine outside of the VPC, a public IP address or an elastic IP address will have to be assigned to the EC2 instance. In this case, we will assign a regular public IP address for demonstration purposes. Please note regular public IP address may change when the instance is restarted.

![aws-ec2-mono-instance-details](https://github.com/user-attachments/assets/e275cfb2-4945-42f4-b927-1efc0bdeb00f)

1. On the EC2 instance details page, click "Actions" button, >Networking >Manage IP Addresses. Select the network interface represented by "eth0" and enable auto-assign public IP. Then click save and confirm the changes. The EC2 instance should now be assigned a public IP address.

![aws-ec2-instance-publicip](https://github.com/user-attachments/assets/77204e88-1925-4049-8f33-4b7b7bde1cb3)

Please ensure the permissions on your private key is secure else there may be issue with remotely accessing the instance.

1. Remotely log into the instance via ssh using the previously downloaded key pair.
   ```bash
   ssh -i <private key location> username@serverIP
   ```
2. Update virutal machine and upgrade packages
      ```bash
      sudo apt update -y
      sudo apt upgrade -y
      ```
      ![aws-ec2-instance-update](https://github.com/user-attachments/assets/19fce26a-e460-4e12-b0fd-7edf7dc35731)

### Preparing LAMP Stack - Linux, Apache, MySQL, PHP

### Installing Webserver - Apache 2
   ```bash
   sudo apt install apache2 -y
   ```
### Installing Database - MySQL
   ```bash
   sudo apt install mysql-server -y
   ```
### Installing PHP and PHP modules
   ```bash
   sudo apt install php libapache2-mod-php php-mysql php-xml php-mbstring php-zip php-intl php-gd php-curl php-soap -y
   ```
### Configure MySQL
Secure the MySQL installation by running
   ```bash
   sudo mysql_secure_installation
   ```
Follow the prompts to secure your MySQL server (set a root password, remove anonymous users, disallow root login remotely, remove the test database, and reload privileges).

Log into MySQl as the root user:
   ```bash
   sudo mysql -u root -p
   ```
Create a database and user for moodle
   ```bash
   CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'admin'@'localhost' IDENTIFIED BY '********';
   GRANT ALL PRIVILEGES ON moodle.* TO 'admin'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```
### Download and Setup Moodle
Download the latest version of Moodle
```bash
   sudo wget https://download.moodle.org/download.php/direct/stable405/moodle-latest-405.tgz
```

Extract the moodle archive
```bash
   sudo tar -xvzf moodle-latest-405.tgz
```

Move the moodle files to the webroot directory
```bash
sudo mv moodle /var/www/
```

Create a Moodle data directory and set file permissions (where Moodle will store its files):
```bash
sudo mkdir /var/www/moodledata
sudo chown -R www-data:www-data /var/www/moodledata /var/www/moodle
sudo chmod -R 755 /var/www/moodledata /var/www/moodle
```

### Configure Apache 2 for Moodle
Create a new apache configuration file for moodle
```bash
sudo nano /etc/apache2/sites-available/moodle.conf
```

### Important notice regarding Moodle routing on version 4.5+
Since Moodle 4.5, a Routing Engine is included in Moodle. This needs to be configured to handle requests using the "FallbackResource" directive:
More could be found on this here - https://docs.moodle.org/405/en/Apache

Replicate the following strucure:
```bash
 ServerAdmin admin@example.com

DocumentRoot /var/www/moodle
Servername demo.example.com
<Directory /var/www/moodle>
        #Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all granted
    FallbackResource /moodle/r.php
</Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined

```

Enable the moodle site and rewrite module, then restart
```bash
sudo a2ensite moodle.conf
sudo a2enmod rewrite
```

Remember to disable the default apache site
```bash
sudo a2dissite 000-default.conf
```
Set PHP max_input_vars to 5000 and remove ";".
```bash
sudo nano /etc/php/8.3/apache2/php.ini
```

Restart apache
```bash
sudo ssytemctl restart apache2
```
### Complete Moodle Setup via Browser
![moodle-setup-page](https://github.com/user-attachments/assets/434be730-baf7-4c3c-ac47-c64ded2cdac1)
Open a browser and navigate to your server's domain or IP address.

Follow the on-screen installation wizard to configure Moodle. Provide the database credentials and configure the necessary settings.

Confirm paths
![moodle-paths](https://github.com/user-attachments/assets/374ef2e4-5130-4a34-961f-de7f765580d9)

Choose database driver
![moodle-database](https://github.com/user-attachments/assets/47f758eb-2a18-4b86-a0e8-ac785548701b)

Database settings
![moodel-database-settings](https://github.com/user-attachments/assets/3f74a58a-da17-4ecd-8c64-1e5705f32764)

Moodle copyright notice
![moodle-copyright](https://github.com/user-attachments/assets/7df0c049-cd22-450b-b744-a99b035ab517)

Moodle server checks
![moodle-server-checks](https://github.com/user-attachments/assets/112dd500-1646-4a4d-a8c3-1d29ea70b3ae)


During the installation process, Moodle will guide you through setting up the administrator account and configuring your site's main settings.

### Successful installation
We have now successfully installed Moodle LMS on a Linux Ubuntu 24.04 LTS server. You can now log into Moodle and manage your learning environment.
![moodle-complete](https://github.com/user-attachments/assets/6ef3161a-5390-4220-937d-44332dbdcff0)





