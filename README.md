# Install-WordPress-on-an-EC2-Machine-with-RDS-as-the-Database
Installing WordPress on an EC2 Machine using RDS (MYSQL) as the Database.
**Installing WordPress on an EC2 Machine with RDS as the Database**

**Sign Up or Log In to AWS**

- Create an AWS free-tier account if you don't already have one. If you do, simply log in to your existing account.

**Configure RDS Database**

- In the AWS Console, navigate to the Amazon RDS Dashboard by clicking on "Services" and then selecting "RDS" under the Database section.

- Create a New DB Instance:
   - Choose the "Standard" database creation method.
   - Select the MySQL database engine.

- Choose a Use Case:
   - Select the use case that best matches your database requirements (e.g., "Dev/Test," "Production," or "Free" for the free tier).

- Specify DB Details:
   - Choose Engine Version.
   - Configure availability and durability options (for "Dev/Test" or "Production" apps).
   - Provide a database name in "DB instance identifier" (e.g., "wordpress").
   - Specify the Database master username and password.
   - Choose the instance type configuration from Standard, Memory optimized, or Burstable classes depending on your workload (only "Burstable classes" in the free tier).
   - Configure Strorage type and volume size minimum is 20 GiB.

- Connection Settings:
   - Select "Don’t connect to an EC2 compute resource, For now."
   - Choose the network type as "IPV4."
   - Select the VPC in which you want your database to be launched (note that this cannot be changed while the database is running).
   - Set Public access to "No."
   - Create a new VPC security group and name it (e.g., "wp_database_sg").
   - Leave Availability Zone, RDS Proxy, and Certificate authority settings as is.
   - Specify the Database authentication password only.

- Additional Configuration:
   - Provide an Initial database name (e.g., "wordpress").
   - Configure your automated backups and leave the rest as default.

- Review your configuration and create the RDS instance. It will take a few minutes to provision.
- Once the RDS instance is created, note down the "Endpoint" value from the RDS dashboard.

**Setting Up an EC2 Instance**

   - Navigate to the EC2 service within the AWS Console.
   - Ensure that the IAM user associated with your account has the necessary EC2 permissions to avoid encountering any errors.

**Selecting Region and Availability Zone**

   - Choose the specific AWS Region according to your project's requirements.

**Instance Configuration**

   - Under the "Instances" section, select "Launch Instances."
   - Provide a meaningful name for your instance and optionally, add tags for resource organization.
   - Select the Amazon Machine Image (AMI), choose "Amazon Linux 2".
   - Choose the instance type based on your server's requirements. (For the AWS free tier, options like "t2.micro" or "t3.micro" are suitable, depending on your region.)

**Key Pair Creation**

   - Create a new key pair. This key pair represents your public key and is required for SSH access to your EC2 machine. Ensure the utmost care in protecting this key. If you already have a key, you can choose to use it.

**Networking Configuration**

   - Select the Virtual Private Cloud (VPC) and Subnet. Amazon provides default VPC and Subnet options in each region, but you can also opt for custom configurations.
   - Assign an auto-assigned IP address.

**Security Group Configuration**

   - Create a new security group or use an existing one. Ensure that this security group has SSH, HTTP, and HTTPS ports open to the public (0.0.0.0).

**Storage Configuration**

   - Configure Elastic Block Store (EBS) volumes, specifying type and size (e.g., gp2/gp3 for free tier, depending on your region).
   - Optionally, encrypt the EBS volume with a Key Management Service (KMS) key, ensuring that you have the necessary KMS permissions.

**Advanced details (User Data)** 

- Paste the following script into the user data section.

```bash
#!/bin/bash
sudo yum update -y
sudo yum install httpd php mariadb-server -y
sudo amazon-linux-extras install php7.2 -y
sudo yum install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
sudo systemctl start httpd mariadb
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
sudo rm latest.tar.gz
sudo mv wordpress /var/www/
sudo chown -R apache /var/www/wordpress
sudo chmod -R 775 /var/www/wordpress
sudo systemctl enable httpd mariadb
```
- Launch the Instance.

**Allowing inbound traffic from the security group of EC2 instance to RDS Database (usually 3306).**
- Head to EC2 then Security Groups and select the RDS Security Group that is created while launching the RDS database instance (e.g., "wp_database_sg").
- Inbound rules and Edit inbound rules.
- Delete any Previous rule and add new:
   - Type: MYSQL/Aurora.
   - Source: Custom.
   - Choose your EC2 instance security group.
- Save the rule.

**Setting Up SSH to Access Your Server**

   - Open your preferred SSH client.

   - Locate the private key file associated with your instance, which is typically in the format KEYXXX.pem.

   - Run the following command to secure your private key(change"KEYXXX.pem" with your key name in the command).

     ```bash
     chmod 0400 KEYXXX.pem
     ```
   - Use the following SSH command to access your server (change "KEYXXX.pem" with your KEY and "3-XXX-XX-235" with your server public IP address in the command).

     ```bash
     ssh -i KEYXXX.pem ec2-user@3-XXX-XX-235
     ```
**Accessing Your RDS Database**
- In your terminal, enter the following command to set an environment variable for your MySQL host and connect to your database. Be sure to replace "<your-endpoint>" with the hostname 
  of your RDS instance and enter your master username and password. ("wordpress" in command is the database name) :
```bash
mysql -h <your-endpoint> -u admin -p wordpress
```
- Replace 'wpuser' and 'wppassword' with a secure username and password in the command:

```sql
CREATE USER 'wpuser' IDENTIFIED BY 'wppassword';
GRANT ALL PRIVILEGES ON wordpress.* TO wpuser;
FLUSH PRIVILEGES;
Exit
```
**Making Wordpress default**

   - Update the httpd configuration file to set ss as the default DocumentRoot:

     ```bash
     sudo nano /etc/httpd/conf/httpd.conf
     ```

   - Change:

     ```
     DocumentRoot /var/www/html
     ```

     to:

     ```
     DocumentRoot /var/www/wordpress/
     ```

   - Restart httpd:

     ```bash
     sudo systemctl restart httpd
     ```

Done, Now Access Your ss Installation & Set it up using Server Public IP Address and in Database Host Paste the RDS Database Endpoint. (Example: http://3-XXX-XX-235 use http not https)

**Installing SSL Certificate for ss**

1. **Adding Your Website's DNS Name to the Configuration File**

   - Edit the httpd configuration file:

     ```bash
     sudo nano /etc/httpd/conf/httpd.conf
     ```

   - Add your website's DNS name by pasting the following at the end of file and replace ServerAdmin, ServerName, ServerAlias with your values:

     ```
      <VirtualHost *:80>
      ServerName myxxxxxx.com
      DocumentRoot /var/www/wordpress
      </VirtualHost>
     ```
   - Restart httpd:

     ```bash
     sudo systemctl restart httpd
     ```
2. **Installing SSL/TLS Certificate Using Let's Encrypt Certbot**

   - Install Epel, Certbot and the Certbot Apache plugin:

     ```bash
     sudo amazon-linux-extras install epel -y
     ```
     ```bash
     sudo yum install python2-certbot-apache -y
     ```
       
   - Run Certbot to obtain and configure the SSL certificate:

     ```bash
     sudo certbot --apache -d myxxxxxxxx.com
     ```

   - Follow the prompts and provide your email.

Congratulations! Your ss website is now set up securely with SSL/TLS encryption. Enjoy your website.
