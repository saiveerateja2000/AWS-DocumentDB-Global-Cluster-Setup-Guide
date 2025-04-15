
# AWS DocumentDB Global Cluster Setup Guide

This guide provides a step-by-step process for setting up an Amazon DocumentDB cluster in the N. Virginia region and then replicating it to the Ohio region. The process includes VPC creation, subnet setup, security groups, cluster creation, replication setup, and verification.

## **Step 1: Create VPC in N. Virginia (us-east-1)**
1. Open **AWS Console** → **VPC**
2. Click **Create VPC**
   - Name: `svt-vpc-virginia`
   - IPv4 CIDR Block: `10.5.0.0/16`
   - Tenancy: Default
   - Click **Create VPC**

## **Step 2: Create Subnets in N. Virginia**
1. Navigate to **Subnets**
2. Click **Create subnet**
   - VPC: `svt-vpc-virginia`
   - Name: `svt-public-subnet-1`
   - Availability Zone: `us-east-1a`
   - CIDR Block: `10.5.1.0/24`
   - Click **Create Subnet**
3. Repeat the process for:
   - `svt-public-subnet-2` in `us-east-1b` with `10.5.2.0/24`
   - `svt-private-subnet-1` in `us-east-1a` with `10.5.3.0/24`
   - `svt-private-subnet-2` in `us-east-1b` with `10.5.4.0/24`

## **Step 3: Create Internet Gateway**
1. Navigate to **Internet Gateways**
2. Click **Create Internet Gateway**
   - Name: `svt-igw-virginia`
   - Click **Create**
3. Attach to `svt-vpc-virginia`

## **Step 4: Create Route Tables**
1. Navigate to **Route Tables**
2. Click **Create Route Table**
   - Name: `svt-public-rt`
   - VPC: `svt-vpc-virginia`
   - Click **Create**
3. Select `svt-public-rt` → Click **Edit Routes** → Add:
   - Destination: `0.0.0.0/0`
   - Target: `svt-igw-virginia`
4. Associate **Public Subnets** with `svt-public-rt`
5. Create a **Private Route Table** for private subnets

## **Step 5: Create Security Groups**
1. Navigate to **Security Groups**
2. Click **Create Security Group**
   - Name: `svt-docdb-sg`
   - VPC: `svt-vpc-virginia`
3. Add Inbound Rule:
   - Type: **MongoDB (TCP 27017)**
   - Source: Custom (Specify EC2 Security Group or IP)
   - Click **Create**

## **Step 6: Create DocumentDB Cluster in N. Virginia**
1. Navigate to **Amazon DocumentDB**
2. Click **Create Cluster**
   - Name: `svt-docdb-cluster-virginia`
   - Instance class: db.r5.large
   - Replica count: 2
   - VPC: `svt-vpc-virginia`
   - Subnets: Select private subnets***
   - Security Group: `svt-docdb-sg`
   - Click **Create Cluster**

## **Step 7: Create an EC2 Instance in N. Virginia**
1. Navigate to **EC2** → **Launch Instance**
2. Name: `svt-ec2-virginia`
3. AMI: Amazon Linux 2
4. Instance Type: t2.micro
5. VPC: `svt-vpc-virginia`
6. Subnet: `svt-public-subnet-1`
7. Security Group: Allow **SSH (22)** and **MongoDB (27017)**
8. Key Pair: Create or select an existing one
9. Click **Launch Instance**

## **Step 8: Install Mongo Shell on EC2**
1. SSH into the EC2 instance
2. Run the following commands:
   ```sh
   sudo yum install -y amazon-linux-extras
   sudo amazon-linux-extras enable corretto8
   sudo yum install -y mongodb-mongosh
   ```
3. Connect to DocumentDB:
   ```sh
   mongosh "mongodb://svt-docdb-cluster-virginia.cluster-xyz.us-east-1.docdb.amazonaws.com:27017/" --tls --username admin --password YourPassword
   ```

## **Step 9: Create VPC in Ohio (us-east-2)**
1. Repeat Steps **1-5** but use CIDR **10.6.0.0/16**
2. Create:
   - VPC: `svt-vpc-ohio`
   - Subnets: `svt-public-subnet-ohio`, `svt-private-subnet-ohio`
   - Internet Gateway: `svt-igw-ohio`
   - Route Tables and Security Groups

## **Step 10: Create VPC Peering Between N. Virginia & Ohio**
1. Navigate to **VPC Peering**
2. Click **Create Peering Connection**
   - Name: `svt-peer-virginia-ohio`
   - Requester: `svt-vpc-virginia`
   - Accepter: `svt-vpc-ohio`
3. Accept the Peering Request in **Ohio VPC**
4. Update Route Tables:
   - Virginia Route Table → Add `10.6.0.0/16 → VPC Peering`
   - Ohio Route Table → Add `10.5.0.0/16 → VPC Peering`

## **Step 11: Create DocumentDB Global Cluster**
1. Navigate to **Amazon DocumentDB**
2. Click **Global Clusters** → **Create Global Cluster**
3. Select `svt-docdb-cluster-virginia`
4. Click **Add Region** → Select `Ohio`
5. Choose Instance Type and Security Group
6. Click **Create Global Cluster**

## **Step 12: Validate Replication**
1. SSH into **Ohio EC2**
2. Install MongoDB shell (repeat **Step 8**)
3. Connect to **Ohio Cluster**:
   ```sh
   mongosh "mongodb://svt-docdb-cluster-ohio.cluster-xyz.us-east-2.docdb.amazonaws.com:27017/" --tls --username admin --password YourPassword
   ```
4. Check Databases:
   ```sh
   show dbs;
   ```
5. Check Collection Data:
   ```sh
   use neuron;
   db.users.find().pretty();
   ```

## **Step 13: Conclusion**
Your **Amazon DocumentDB Global Cluster** is now set up between **N. Virginia and Ohio** with full replication. 🚀

