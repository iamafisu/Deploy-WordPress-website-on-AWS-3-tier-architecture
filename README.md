# WordPress Setup on Amazon Linux 2

This repository contains a set of scripts to simplify the installation and configuration of a WordPress website on an EC2 instance using Amazon Linux 2. These scripts automate the process of setting up the necessary components, including Apache web server, PHP 7.4, MySQL 5.7, EFS and WordPress. 

## Prerequisites

Before you begin, ensure that you perform the following:

- Create a VPC with 4 private and 2 public subnets in two availability zones.
- create an Internet Gateway and attach it to the VPC.
- create Route Tables (public & private) and associate the public subnets to the public route table and private subnets to the private route table.
- Create 5 Security Groups (ALB_SG, SSH_SG, WEBSERVER_SG, DATABASE_SG & EFS_SG)
  ALB_SG
    (Open port = 80 & 443 from/source = 0.0.0.0/0)
  SSH_SG
    (Open port = 22 from/source = MyIP)
  WEBSERVER_SG
    (Open port = 80 & 443 from/source = ALB_SG
    Open port = 22 from/source = SSH_SG)
  DATABASE_SG
    (Open port = 3306 from/source = WEBSERVER_SG)
  EFS_SG
    (Open port = 2049 from/source = WEBSERVER_SG, EFS_SG
    Open port = 22 from/source = SSH_SG)
- Create RDS database: create subnet group
- Create Elastic File System (EFS) with EFS mount target in the private subnet.
- Create an auto-scaling group with a launch template to launch EC2 Instances.
- Create an Application Load Balancer: create a target group for the ALB.
- Register a new domain or use an existing domain with Route 53.
- Request for an SSL certificate using the Certificate Manager.


## Note
- Enable DNS hostname in the vpc
- Enable auto-assign public IPv4 address in the public subnets.

## Steps to Set Up WordPress

Follow the steps below to set up a WordPress website using the provided scripts:

### 1. Mount EFS to HTML Directory

```bash
sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0ee96f135f9541107.efs.us-east-1.amazonaws.com:/ /var/www/html
```

### 2. Install Apache

```bash
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd
sudo systemctl start httpd
```

### 3. Install PHP 7.4

```bash
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
```

### 4. Install MySQL 5.7

```bash
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld
```

### 5. Set Permissions

```bash
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html
```

### 6. Download WordPress Files

```bash
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/
```

### 7. Create the wp-config.php File

```bash
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```

### 8. open the wp-config.php File

```bash
nano /var/www/html/wp-config.php
```

### 9.  Add and edit the wp-config.php File with the script

```bash
/* SSL Settings */
define('FORCE_SSL_ADMIN', true);

// Get true SSL status from AWS load balancer
if(isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
  $_SERVER['HTTPS'] = '1';
}
```

```bash
DATABASE NAME: "ENTER YOUR DATABASE NAME"
USERNAME: "ENTER YOUR USERNAME"
PASSWORD: "ENTER YOUR DATABASE PASSWORD"

```

### 10. Securing the website
- Secure the website by adding the HTTPS listener in the ALB.

  
### 11. Restart the Web Server

```bash
service httpd restart
```

## NFS Mounting

Before running these scripts, make sure you have set up an Amazon EFS filesystem and have the necessary NFS mount target details configured in the `mount` command within the script.

## Disclaimer

Please note that these scripts are meant to streamline the installation process on Amazon Linux 2 instances. Make sure to review and customize configurations as needed, especially when dealing with production environments or security considerations.

For any questions or troubleshooting, feel free to reach out to me and you can also refer to official AWS and software documentation.

## License

This project is licensed under the [MIT License](LICENSE).

For any questions or inquiries, please contact afisu.g@hotmail.com. Happy wordpress deploying!
