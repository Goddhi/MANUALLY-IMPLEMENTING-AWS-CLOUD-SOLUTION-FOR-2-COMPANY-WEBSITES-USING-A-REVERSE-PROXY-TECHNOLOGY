# MANUALLY-IMPLEMENTING-AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOG

## Table of Contents

Introduction
Starting Off Your AWS Cloud Project
SET UP A VIRTUAL PRIVATE NETWORK (VPC)
Creating the Subnets
Creating the Route Tables
Creating an Elastic IP
Creating a NAT Gateway
Creating the Security Groups
Assigning Certificate and creating Hosted Zone
Creating the Elastic File System (EFS)
Creating the Relational Database Service (RDS), Key Management Service (KMS) and Subnet Groups
Creating our Resources
Bastion AMI installation
Nginx AMI installation
Webserver AMI installation
Creating our AMI images
Creating our Target groups
Creating our Load Balancers
Creating the Autoscaling Groups
Creating the Route53 records

## Introduction

In this project, you will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

## INFRASTRUCTURE ARCHITECTURE DIAGRAM

![infrastructure-architecture](/Images/resource-architecture.png)

### Starting Off Your AWS Cloud Project

There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit [Watch How To Do This Here](https://youtu.be/9PQYCc_20-Q)

- Create an AWS Master account. (Also known as Root Account).
- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

Note: For a newly created account you might need to configure AWS service quota to allow you to create more accounts on the root account. [Read how to do this here](https://aws.amazon.com/premiumsupport/knowledge-center/organizations-account-exceeded/#:~:text=Resolution,Line%20Interface%20(AWS%20CLI).)

- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)

- Move the DevOps account into the Dev OU.
- Login to the newly created AWS account using the new email address.

2. Create a free domain name for your fictitious company at Freenom domain registrar [here](https://www.freenom.com/).

Create a hosted zone in AWS, and map it to your free domain from Freenom. This gives a guide on how to do this

3. Create a hosted zone in AWS, and map it to your free domain from Freenom. [This gives a guide on how to do this](https://youtu.be/IjcHp94Hq8A)

### SET UP A VIRTUAL PRIVATE NETWORK (VPC)

A VPC is a virtual network that you can define within AWS. It is logically isolated from other virtual networks in the AWS Cloud. You can launch your AWS resources, such as Amazon EC2 instances, into your VPC. You can specify an IP address range for the VPC, add subnets, associate security groups, and configure route tables. You can also create an Internet gateway and attach it to your VPC so that instances have access to the Internet. For more information, see [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).

We would be using the architecture diagram above for this project.

- Create a VPC

output: 

![vpc](Images/vpc.png)

We need to ensure that dns hostnames are enabled for our VPC. This is to ensure that our EC2 instances can resolve each other using their private IP addresses. To do this, go to the VPC dashboard, select your VPC, and click on the Actions button. Select Edit DNS Hostnames and enable it.

![pc-settings](Images/pc-settings.png)

- Create an internet gateway and attach it to the created vpc.

![internet-gateway](Images/igw.png)

- Then attach the internet gateway to the VPC.

![attach-igw](Images/attach-igw.png)

### Creating the Subnets

- The next thing is to create the subnets. We would use ipinfo to get the IP address ranges for the different regions. You can decide how you intend to assign the IP address ranges to the subnets. Here I would be using even-number addresses for public subnets and odd-number IP addresses for private subnets. We would use the IP address ranges for the different regions to create the subnets. We would create 6 subnets for the VPC. Two public subnets and four private subnets The subnets would be in the us-east-1 availability zone. As we would be using us-east-1a and us-east-1b.

Public subnet 1 and Public subnet 2 
![AWS public subnet 1 and 2](Images/public1-2.png)


Private subnets 1 2 3 4
![all-priate-subnets](Images/priate-subnets.png)

### Creating the Route Tables

- We then move on to creating the route tables. Here, we would be creating two route tables. A public and private route table.

Public and Private rtb
![public-priate-rtb](Images/rtb.png)

- Now we associate the route tables with their corresponding subnets. Navigate to the subnet associations after selecting the route table and click on edit associations. Select the subnets you want to associate with the route table and click save.

Associate public route table with public subnets
![Associate-pub-sub-rtb](Images/associate-pub-rtb.png)

Associate private route table with private subnets
![Associate-pri-sub-rtb](Images/associate-pri-sub-rtb.png)

- Now we need to edit the routes of the route tables.

  - For the public route table, we would add a route to the internet gateway. This is to allow the public subnets to have access to the internet. To do this, click on the route table, select routes, and click on edit routes. Add a new route with the destination as '0.0.0.0/0' and the target as the internet gateway, this would bring up the internet gateway drop down. Select the internet gateway you created earlier and click save.

![edit-route-for-pub-subnet](Images/public-sub-rtb.png)

### Creating an Elastic IP

- Before we create the private route, we need to create an elastic ip address.

![aws-elastic-ip](Images/aws-elp.png)

### Creating a NAT Gateway

- Then we need to create NAT gateways.
![nat-gat](Images/nat-gat.png)

Now let's navigate back to our route tables and select the private route table. Click on routes and edit routes. Add a new route with the destination as '0.0.0.0/0' and the target as the NAT gateway. This would bring up the NAT gateway dropdown. Select the NAT gateway you created earlier and click save.
![nat-pri-subnet](Images/nat-pri-sub.png)

### Creating the Security Groups

- Now we need to create security groups for all the application resources. It is important to allocate the security groups we create to the vpc created earlier.
  - Create a security group for our External Application Load Balancer (ALB). This would be a public security group. This is to allow the ALB to receive traffic from the internet. We would allow traffic on port 80 and 443. This is to allow traffic from the internet to the ALB.
  
![scg-https and http ](Images/scg-http.png)

Create a security group for our Bastion host. This is to allow us to access the Bastion host from the internet. We would allow traffic on port 22. And we would restrict the source to our IP address.

![scg-bastion](Images/scg-bastion.png)

- Create a security group for the Nginx reverse proxy server. It is important to note that from the architecture diagram, the Nginx reverse proxy server interacts with the application load balancer (ALB) and not the bastion. So the source of the traffic would be the ALB security group. We would allow traffic on port 80 and 443. And we would restrict the source to the ALB security group. For emergencies, we need to allow ssh traffic from the bastion server. So we would allow traffic on port 22 and restrict the source to the bastion security group.

![securty group for rerse proxy](Images/scg-reerse-proxy.png)

- We also need to create a security group for the internal application load balancer (ALB). This is to allow the ALB to receive traffic from the Nginx reverse proxy server. We would allow traffic on port 80 and 443. And we would restrict the source to the Nginx reverse proxy server security group.

![security group for ingternal ](Images/aws-vpc-security-group-internal-alb.png)

- We also need to create a security group for our web servers and the traffic should come only from our internal application load balancer (ALB). We would allow traffic on port 80 and 443. And we would restrict the source to the internal ALB security group. And also ssh from the bastion server only.

![scg for webserer](Images/aws-vpc-security-group-webserver.png)

- We also need to create a security group for the Data Layer. This is to allow the web servers to connect to the database. We would allow traffic on port 3306. And we would restrict the source to the web server security group.

![scg for datalayer](Images/aws-vpc-security-group-database.png)

### Assigning Certificate and creating Hosted Zone

- We need to create our certificates, before this you need to get a domain and transfer it to AWS Route 53. [Here's a guide on how to transfer your domain](https://www.youtube.com/watch?v=3lWo3ovMhTA)

- Now create a hosted zone in route 53 bearing the name of your domain. Then create a record set for the domain. This is to allow the domain to resolve to the ALB.

- Navigate to the certificate manager and create a certificate for your domain. This is to allow the ALB to use the certificate for the domain.

  - In creating the certificate, you would be asked to choose the domain name. Select the domain name you created earlier, ensure you use a wildcard domain name. This is to allow the certificate to be used for all subdomains. Then click on next.
Then click on the certificate, in the domain section you would be asked to create records in route 53. Click on create records in route 53. This is to allow the certificate to be validated.

### Creating the Elastic File System (EFS)

- Now we proceed to create the Amazon Elastic File system (EFS). This is to allow the web servers to share files. Navigate to the EFS dashboard and click on create file system. Select the vpc that was earlier created and click on customize. Then click next. Here you would be asked to create mount targets. Select the subnets you want to create the mount targets in (private subnets 1&2). Remember to change the security group to datalayer security group. This is to allow the web servers to connect to the EFS. Then click next. Then click on create file system.


![img of efs](Images/aws-vpc-efs.png)


![img  of efs network](Images/aws-vpc-efs-network.png)

Note: According to our architecture diagram, our web servers are in private subnets 1 & 2. As they would be the ones that need to mount to the EFS.

- In our filesystem, we need to create access points. One for our tooling and the other for wordpress server. Navigate to the EFS dashboard and select the file system you created earlier. Then click on access points. Click on create an access point.

  - specify the name of the access point.
  - specify the path of the root directory. This is to allow the access point to be used for a specific directory.

  - select the POSIX user:
    - user ID : 0
    - group ID : 0

  - select the root directory creation permissions:
    - owner ID : 0
    - group ID : 0
    - permissions: 0755

Repeat the above steps to create another access point for the tooling server. 

![efs-access point](Images/efs-access-point.png)

Note: The purpose of creating two access points is to prevent the case of files overwriting each other when the WordPress server and the tooling server are both writing to the same access point.

### Creating the Relational Database Service (RDS), Key Management Service (KMS) and Subnet Groups (Optional)

  - Let's create the KMS key. Navigate to the KMS dashboard and click on create key.

     - select the key type as symmetric.
     - select the key usage as encrypt and decrypt.
     - click on next.
     -  select a name for the key.
     -  select the key administrator.(you can select your own account)
     -  click on next.
     - select the key usage permission. (you can select your own account)
     -  click on finish.

![kms-config](Images/kms-img.png)

- Let's move on with creating the subnet groups. Navigate to the RDS dashboard and click on subnet groups.

  -  click on create db subnet group.
  - specify the name and description of the subnet group.
  - select the vpc you created earlier.
  - select the availability zones that include the subnets you want to     add. In this case, we would select us-east-1a and us-east-1b.
  - select the subnets you want to add. According to the format for assigning IP addresses our private subnets where the RDS would be created are odd-numbered. So we would be selecting the private subnets 3 & 4.
  - click on create.

  ![rds-img](Images/rds-img.png)

  - Now we navigate back to the Amazon rds dashboard and create a database

  -  click on create database.
  -  select the database engine as MySQL.
  -  select the latest engine version.
  -  select the free tier template. The only downside to this is that we are unable to encrypt the database. As using the production template requires a huge amount of money.
  -  select your db instance identifier.
  -  select the master username and password. Here we used ACSadmin and admin12345 respectively.
  -  select the vpc you created earlier.
  -  For the public access, we would select no. This is to prevent the database from being accessed publicly.
  -  For the vpc security group, we would select the datalayer security group created much earlier.
  -  In the additional configuration, we would specify the initial database name as test
  -   Then click on create database

  ![rds-img-2](Images/rds-img2.png)

  ### Creating our Resources

- Target group

- Launch template

- Load balancer

- Auto scaling group

- Create our launch template. We would be creating 3 instances that we would use as our template.

    - Navigate to the EC2 dashboard and click on ec2 instances.
    - Click on launch instance.
    - Select the AMI you want to use. In this case, we would be using the RedHat linux 8 AMI.
    - Select the instance type. In this case, we would be using the t2.micro.
    - Configure the instance and change the number of instances to 3.
    - Configure the security group. Here we would be creating a new security group. We would be allowing traffic all traffic.
    - Then click on launch.

![img-instances](Images/aws-vpc-ec2-launch-instances.png)

- Rename the 3 instances
    - bastion
    - nginx
    - webserver

![img-instances](Images/img-instances.png)
  
- Now we ssh into our bastion host to install the necessary packages using mobaxterm. 

### Bastion AMI installation

- SSH into the bastion server
  - switch to root user
  - install the necessary packages

```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 

yum install wget vim python3 telnet htop git nano mysql net-tools chrony -y 

systemctl start chronyd 

systemctl enable chronyd

```

### Nginx AMI installation

- SSH into the nginx server
  - switch to root user

```
sudo su -
```
   - install the necessary packages
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet nano htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
- configure selinux policies for the webservers and nginx servers

```
setsebool -P httpd_can_network_connect=1

setsebool -P httpd_can_network_connect_db=1

setsebool -P httpd_execmem=1

setsebool -P httpd_use_nfs 1
```
- We will install amazon EFS utils for mounting the target on the Elastic file system

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```
- Seting up self-signed certificate for the nginx instance

```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

```
  - The openssl command will prompt you to enter the following information:
      - Country Name (2 letter code) [AU]: NG
      - State or Province Name (full name) [Some-State]: Lagos
      - Locality Name (eg, city) []: Ikeja
      - Organization Name (eg, company) [Internet Widgits Pty Ltd]: ACS
      - Organizational Unit Name (eg, section) []: IT
      - Common Name (e.g. server FQDN or YOUR name) []: <nginx-server private dns>
      - Email Address []:
```
- To check if our certificate has been generated run the following command
```
sudo ls /etc/ssl/certs/
```
As we would be looking for ACS.crt.



































