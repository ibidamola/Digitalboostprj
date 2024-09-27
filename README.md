# DIGITAL BOOST WEBSITE

## Table of Contents
- [DIGITAL BOOST WEBSITE](#digital-boost-website)
  - [Table of Contents](#table-of-contents)
  - [Project Overview](#project-overview)
  - [VPC Setup](#vpc-setup)
    - [IP Address Range Definition](#ip-address-range-definition)
    - [Steps to Create VPC](#steps-to-create-vpc)
    - [Route Table Configuration](#route-table-configuration)
    - [Subnet and NAT Gateway Setup](#subnet-and-nat-gateway-setup)
  - [Public and Private Subnet with NAT Gateway](#public-and-private-subnet-with-nat-gateway)
    - [Steps to Create Public Subnet](#steps-to-create-public-subnet)
    - [Steps to Create Private Subnet](#steps-to-create-private-subnet)
    - [Steps to Create NAT Gateway](#steps-to-create-nat-gateway)
  - [AWS MySQL RDS Setup](#aws-mysql-rds-setup)
    - [Steps to Create RDS Instance](#steps-to-create-rds-instance)
    - [Steps to Configure RDS Security Groups](#steps-to-configure-rds-security-groups)
    - [Steps to Connect WordPress to RDS](#steps-to-connect-wordpress-to-rds)
  - [EFS Setup for WordPress Files](#efs-setup-for-wordpress-files)
    - [Steps to Create EFS](#steps-to-create-efs)
    - [Steps to Mount EFS on EC2](#steps-to-mount-efs-on-ec2)
  - [Application Load Balancer and Auto Scaling](#application-load-balancer-and-auto-scaling)
    - [Steps to Create ALB](#steps-to-create-alb)
    - [Steps to Configure ALB Listener Rules](#steps-to-configure-alb-listener-rules)
    - [Steps to Integrate ALB with Auto Scaling](#steps-to-integrate-alb-with-auto-scaling)
  - [Conclusion](#conclusion)

---

## Project Overview

This project involves creating a highly scalable WordPress website on AWS for a digital marketing agency, "DigitalBoost." The infrastructure includes a VPC, subnets, NAT gateways, MySQL RDS, EFS, ALB, and Auto Scaling groups to ensure high availability and fault tolerance.

---

## VPC Setup

### IP Address Range Definition

We define the VPC with a CIDR block of 10.0.0.0/16.

**Dummy Image: VPC CIDR Block Creation**  
![VPC CIDR Creation](path/to/your/image)

### Steps to Create VPC
1. In the AWS Management Console, search for **VPC**.
2. Click on **Create VPC**.
3. Enter the Name Tag for the VPC (e.g., `wordpress-vpc`).
4. Specify the IPv4 CIDR block as `10.0.0.0/16`.
5. Leave the IPv6 CIDR block as default (no IPv6).
6. Set **Tenancy** to default.
7. Click on **Create**.

### Route Table Configuration
1. After the VPC is created, go to the **Route Tables** section.
2. Click **Create Route Table**, and name it (e.g., `public-route-table`).
3. Associate the route table with the public subnet created below.
4. Add a new route to `0.0.0.0/0` and set the target as the Internet Gateway.

**Dummy Image: Route Table Creation**  
![Route Table Creation](path/to/your/image)

### Subnet and NAT Gateway Setup

We create subnets in two Availability Zones (AZs) within the VPC. For each AZ, we will create one public subnet and two private subnets:

- **AZ 1**:
  - Public Subnet: `10.0.0.0/24`
  - Private App Subnet: `10.0.2.0/24`
  - Private Data Subnet: `10.0.4.0/24`
- **AZ 2**:
  - Public Subnet: `10.0.1.0/24`
  - Private App Subnet: `10.0.3.0/24`
  - Private Data Subnet: `10.0.5.0/24`

**Dummy Image: Subnet CIDR Ranges**  
![Subnet CIDR Ranges](path/to/your/image)

Additionally, each public subnet will have a NAT Gateway that connects to an Internet Gateway. The route tables for private subnets in AZ 1 and AZ 2 will route traffic to the respective NAT Gateway in their AZ’s public subnet.

**Dummy Image: NAT Gateway Configuration**  
![NAT Gateway](path/to/your/image)

---

## Public and Private Subnet with NAT Gateway

### Steps to Create Public Subnet
1. Go to the **Subnets** section in the VPC dashboard.
2. Click **Create Subnet**.
3. Enter the Name Tag (e.g., `public-subnet`).
4. Select the VPC created earlier.
5. Set the IPv4 CIDR Block to `10.0.1.0/24`.
6. Click **Create Subnet**.
7. Modify the subnet and enable auto-assign public IP.

**Dummy Image: Public Subnet Creation**  
![Public Subnet](path/to/your/image)

### Steps to Create Private Subnet
1. Follow the same steps as for the public subnet but name it `private-subnet`.
2. Set the IPv4 CIDR Block to `10.0.2.0/24`.
3. Do not enable public IP auto-assignment.

**Dummy Image: Private Subnet Creation**  
![Private Subnet](path/to/your/image)

### Steps to Create NAT Gateway
1. In the VPC dashboard, go to **NAT Gateways**.
2. Click **Create NAT Gateway**.
3. Select the public subnet.
4. Create an Elastic IP if you don’t have one.
5. Click **Create NAT Gateway**.
6. Update the route table for the private subnet to use this NAT gateway for outbound internet access.

**Dummy Image: NAT Gateway Setup**  
![NAT Gateway Setup](path/to/your/image)

---

## AWS MySQL RDS Setup

### Steps to Create RDS Instance
1. In the AWS Management Console, search for **RDS**.
2. Click **Create Database**.
3. Select the **MySQL** engine.
4. Choose **Standard Create**.
5. Set the DB Instance Class to `db.t2.micro`.
6. Enter the Master username and password.
7. Select the VPC and private subnet group.
8. Click **Create Database**.

**Dummy Image: RDS Creation**  
![RDS Creation](path/to/your/image)

### Steps to Configure RDS Security Groups
1. In the VPC dashboard, go to **Security Groups**.
2. Click **Create Security Group**.
3. Name the security group (e.g., `rds-sg`).
4. Add an inbound rule for MySQL/Aurora on port 3306, with the source being the security group of the WordPress EC2 instances.
5. Attach the security group to the RDS instance.

**Dummy Image: Security Group Configuration**  
![Security Group](path/to/your/image)

### Steps to Connect WordPress to RDS
1. SSH into the WordPress EC2 instance.
2. Open the `wp-config.php` file.
3. Add the following code to define the database connection:
    ```php
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'admin');
    define('DB_PASSWORD', 'password');
    define('DB_HOST', '<rds-endpoint>');
    ```
4. Save and close the file.

---

## EFS Setup for WordPress Files

### Steps to Create EFS
1. In the AWS Console, search for **EFS**.
2. Click **Create File System**.
3. Name the EFS (e.g., `wordpress-efs`).
4. Choose the VPC and private subnets.
5. Leave the default settings and click **Create**.

**Dummy Image: EFS Creation**  
![EFS Creation](path/to/your/image)

### Steps to Mount EFS on EC2
1. SSH into the WordPress EC2 instance.
2. Install the NFS client:
    ```bash
    sudo yum install -y nfs-utils
    ```
3. Create a directory to mount the EFS:
    ```bash
    sudo mkdir /mnt/efs
    ```
4. Mount the EFS:
    ```bash
    sudo mount -t nfs4 <file-system-id>:/ /mnt/efs
    ```

**Dummy Image: Mounting EFS**  
![Mounting EFS](path/to/your/image)

---

## Application Load Balancer and Auto Scaling

### Steps to Create ALB
1. In the AWS Management Console, search for **EC2** and go to **Load Balancers**.
2. Click **Create Load Balancer** and select **Application Load Balancer**.
3. Name the load balancer (e.g., `wordpress-alb`).
4. Select the VPC and public subnets.
5. Create a Security Group for the ALB that allows HTTP traffic.
6. Click **Create Load Balancer**.

**Dummy Image: ALB Creation**  
![ALB Creation](path/to/your/image)

### Steps to Configure ALB Listener Rules
1. Go to the **Listeners** tab in the ALB dashboard.
2. Edit the HTTP listener to forward traffic to the target group where WordPress EC2 instances are registered.
3. Click **Create Rule**.

**Dummy Image: Listener Configuration**  
![Listener Configuration](path/to/your/image)

### Steps to Integrate ALB with Auto Scaling
1. Go to the **Auto Scaling Groups** section in the EC2 dashboard.
2. Create an Auto Scaling Group.
3. Select the WordPress EC2 instance as the launch template.
4. Set the minimum, desired, and maximum number of instances.
5. Attach the Auto Scaling Group to the ALB target group.

**Dummy Image: Auto Scaling Integration**  
![Auto Scaling Integration](path/to/your/image)

---

## Conclusion

In this project, we successfully created a scalable WordPress website on AWS using various services including VPC, subnets, NAT gateways, RDS, EFS, ALB, and Auto Scaling. This architecture ensures that the WordPress site is highly available, secure, and can scale efficiently based on demand.

---

