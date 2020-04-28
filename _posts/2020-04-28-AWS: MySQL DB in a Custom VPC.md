Here's something I've been working on that I thought was worth sharing. I set up a private relational database in Amazon's AWS/RDS then connected to it using a virtual ubuntu server set up in Amazon's EC2 service. In this case, the key to connecting the two was creating a virtual private cloud (VPC) where the two resources were spawned.   

Here is a detailed summary of the steps: 

1. create a VPC
2. create public and private subnets 
3. set-up internet gateway
4. set-up route table
5. create security groups
6. set up a private MySQL database in AWS/RDS inside the private subnets within the VPC
7. set up a virtual server in an EC2 Instance in a public subnet within the VPC
8. check to see if a MySQL client is installed on the EC2 instance and install one if necessary
9. connect to the EC2 Instance via SSH protocol
10. connect to the RDS database from the EC2 Instance

the database is in a security group called 'mams-rds-db-sg' and the EC2 instance is in a security group called 'public-vm-sg'. In order to connect to the RDS from EC2 I had to edit the inbound rules for the 'mams-rds-db-sg' to accept MySQL connections from the 'public-vm-sg' security group.

# Big Picture

The most recent lesson I've been working though in my Amazon/AWS/RDS/EC2 skills challenge was focused on setting up a private database in RDS and connecting to it via an EC2 instance. 

The primary purpose of this exercise (as I understand it) is security. By launching the RDS instance in a VPC, it is only accessible by other resources that also reside within that VPC. In this case, access to the database is limited to the EC2 instance that will be spawned in the same VPC. Since access to the EC2 instance is controlled by the use of a unique encrypted private key pair, the database is not exposed to random internet traffic. 

This is notably different from what I did [here in my last post](https://aaronmams.github.io/Connect-to-Amazon-RDS-DB-with-MySQL-Workbench/) where I created a public database. In this case, the inbound rules were set such that anyone (with the proper endpoint and log-in credentials) from any IP address could connect to the database. 

For context, my ultimate goal here is to have an R-Shiny Application hosted on the web. I would like this R-Shiny App to offer some interactive exploration of a database. The database I'm considering will hold private information on individual customers. For this reason, I don't want the database to be publicly accessible. I want access to the database controlled through the R-Shiny App so I can ensure that only aggregated data summaries (that don't expose individual private information) are displayed. I'm nowhere near this end goal yet...but the architecture I'm discussing here is what I believe to be my best shot at operationalizing this longer-term goal.

Now, I'll go through the steps I outlined above 1-by-1:

## 1. Create a Custom VPC

goal: create the 'mams-vpc' virtual private cloud.

* From the AWS Dashboard use the **Services** drop down and search for "VPC". 
* Use the left column to navigate to "Your VPCs"
* Find the call-to-action for "Create VPC"

<img src="/images/aws-home.png" width="400" height="200" />

<img src="/images/aws-create-vpc1.png" width="400" height="200" />

I named my VPC *mams-vpc* and assigned it the CIDR block 192.168.0.0/16. I don't claim any real knowledge of CIDR blocks or how to use them to set up a VPC. I used this block because some [rando on the internet](https://forums.aws.amazon.com/message.jspa?messageID=621473) said it's an ideal block to use for setting up VPCs in AWS.

### 1.1. Create Subnets w/i the VPC

The architecture I am using for this project is to have an RDS database deployed in a private subnet within a VPC and an EC2 Instance that will function as a public-facing webserver. 

The next step is to create these subnets. From the VPC Dashboard within AWS:

* look to the left-rail navigation and click "subnets"
* "Create subnet"

#### 1.1.1. Create the Private Subnet

<img src="/images/aws-create-subnet.png" width="400" height="200" />

#### 1.1.2. Create the Public Subnet

The steps here are the same as for creating the private subnet. 

The only difference between the public and private subnets will be the security features we assign to these subnets later.

At this stage, it also deserves mention that I actually created 2 private subnets 

* mams-priv-sub-1b 
* mams-priv-sub-1c 

This is because I'm going to be deploying my RDS database in two different Amazon Availability Zones (multi-availability zone configuration). This is not critical for the task at hand. 

### 1.2. Internet Gateways

For the public subnet we need an internet gateway so that the AWS resources like EC2 Instances deployed within the public subnet can get internet connectivity.

From the VPC Dashboard left-rail navigation:

* chose "Internet Gateways"
* "Create Internet Gateway"

#### 1.2.1. Attach Internet Gateway to VPC

From the VPC Dashboard:

* find "Internet Gateways" on the left-rail
* click the radio button to the left of the internet gateway that you wish to attach to a VPC
* Using the "Actions" button above the table select "Attach to VPC"

### 1.3. Create Route Tables

From the VPC Dashboard:

* left-rail navigation, "Route Tables"
* "Create Route Table"

Here, I'm creating a route table for the private subnet so I'm giving it the descriptive name, *mams-vpc-priv-rt*.

Here's a moderately interesting aside: when I created my VPC, AWS created a default Route Table for that VPC. Curious readers maybe wondering why then do I need a new route table when a default route table exists? Basically, it's because I created two subnets: one public and one private. Any AWS resources deployed inside the private subnet must not get public IPs or internet connectivity. So that default route table will get assigned to my public subnet while this new route table is going to get assigned to the private subnet.  

Here, you can see that my set-up has a route table (*mams-vpc-priv-rt*) with a local-only route, 

<img src="/images/aws-routetable-priv.png" width="400" height="200" />

and default route table with a local route and a route to the internet gateway.

<img src="/images/aws-routetable-default.png" width="400" height="200" />

#### 1.3.1. Route Table Subnet Association

There's one more step that I need to set up the route tables. I want my private subnets associated with the private (local only) route table and I want my public subnets associated with the public (internet connectivity) route table.

From the Route Table Menu I select the route table that I want to edit then:

* choose the "Subnet Associations" table
* make sure that only the subnets I want associated with this route table are checked
* save changes
* repeat for the other route tables

### 1.4. Create Security Groups

From the VPC Dashboard:

* in the left-rail choose "Security Groups"
* use the blue "Create Security Group" call-to-action to launch the security group wizard
* give the new security group a name and description (mine is *mams-rds-db-sg*)

Next, I'm going to create a security group for the soon-to-be-created EC2 Instance. Same drill as above:

* from the VPC Dashboard find "Security Groups" in the left-rail
* use the "Create Security Group" CTA to launch the security group wizard
* I'm naming this security group *public-vm-sg*

In my architecture, the MySQL Database in AWS/RDS (the *minty hippo* database) will reside in the *mams-rds-db-sg* and the EC2 Instance (still yet to be created) will reside in the *public-vm-sg* security group 


#### 1.4.1. Edit Security Group Features

Since I want my EC2 Instance to be able to communicate with my RDS database, I have to edit the inbound rules in the *mams-rds-db-sg*. Here, I'm setting an inbound rule for *mams-rds-db-sg* of type "MySQL/Aurora" to accept a MySQL connection from resources within the *public-vm-sg* security group. Again, this is because my EC2 Instance will be governed by the *public-vm-sg* security group and I want to connect my EC2 Instance to the MySQL database that is governed by the *mams-rds-db-sg* security group.

To set an inbound rule for the *mams-rds-db-sg* navigate to the VPC Dashbord and find "Security Groups" on the left-rail nav:

* In the top table, I select the security group I want to edit
* Using "Inbound Rules" tab to view the inbound rules
* Use "Edit Rules" to set a new inbound rule

<img src="/images/aws-securitygroup-inboundrules.png" width="400" height="200" />

### 1.5. Create Subnet Groups

I promise this tutorial is rapidly approaching the part where I actually create the database...The last of the foundational steps here is creating the subnet group where the RDS database instance will live. 

From the RDS Dashboard:

<img src="/images/aws-rds-subnetgroup.png" width="400" height="200" />
 
* find "subnet groups" in the left-rail navigation
* above the table displaying existing subnet groups find the CTA button "create database subnet group
* the table below shows the fields needed to create a new subnet group

| field       | value                         |
|-------------|-------------------------------|
| name        | mams-rds-subnet-group         |
| description | subnet group for rds instance |
| VPC ID      | mams-vpc                      |


#### 1.5.1. Assigning subnets within the subnet group

The final step in creating the subnet groups is to assign subnets within the VPC to this subnet group. 

This step is not difficult be does require a little attention. Within the *mams-vpc* VPC, I have 3 subnets defined. Two are private subnets and one is a public subnet. Because this is the subnet group in which I will ultimately provision my private Database Instance, I want to assign ONLY THE TWO PRIVATE SUBNETS to this subnet group.  
