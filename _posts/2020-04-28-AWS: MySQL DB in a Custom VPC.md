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

![](/images/aws-home.png)

![](/images/aws-create-vpc-1.png)

I named my VPC *mams-vpc* and assigned it the CIDR block 192.168.0.0/16. I don't claim any real knowledge of CIDR blocks or how to use them to set up a VPC. I used this block because some [rando on the internet](https://forums.aws.amazon.com/message.jspa?messageID=621473) said it's an ideal block to use for setting up VPCs in AWS.
