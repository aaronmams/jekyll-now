I've been wanting to beef-up my AWS skills for a long time. The main thing that's slowed me down is that we cannot store government data on/in Amazon's AWS ecosystem. This isn't really a hard roadblock, it's just that a lot of my blog content is generated from little programming or data hurdles I encounter at work. 

Well, I picked up a little side project and decided it would be fun to finally try and set up a database on AWS. I suspect this side project will end up generating a few blog posts. This one is dedicated to what should be the relatively simple, discrete task of:

1. setting up a relational database on AWS
2. connecting to that database using MySQL Workbench.

## Goal

The goal in this post is a simple proof-of-concept: get a relational database set up in Amazon's RDS Service and access that database from my laptop using some open source (off-the-shelf) database management software.

## Caveat

I'm a reasonably experienced consumer/user of databases. I have no meaningful experience in database engineering or systems architecture. I'm an analytics guy. My interest in databases is pretty limited to their ability to efficiently deliver me the data I want. This means there are A LOT of gaps in my knowledge. I know next to nothing about security, performance, indexing, and a host of other database topics that are probably really, really important.  

## Resources

1. [Ryan Zhou's instructions](https://medium.com/@ryanzhou7/connecting-a-mysql-workbench-to-amazon-web-services-relational-database-service-36ae1f23d424). This post on Medium is Super Dope. You can probably skip my tutorial entirely and just follow this one.

2. [Amazon's AWS guide to connecting MySQL Workbench to an RDS instance](https://aws.amazon.com/premiumsupport/knowledge-center/connect-rds-mysql-workbench/). I didn't get much out of this. I'm not a database engineer and I don't actually understand a lot of stuff from Amazon's AWS help resources.

3. [More general instruction from Amazon](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.html). Again, I didn't find this super helpful but some people might.

4. [This Udemy Course on AWS RDS Mastery](https://www.udemy.com/course/aws-master-class-databases-in-the-cloud-with-aws-rds/). Not the best Udemy course I've taken...worth the $8 I paid for it but certainly not a penny more. Importantly, their step-by-step guide to setting up and connecting to a MySQL database on Amazon RDS DID NOT work for me. I had to trouble shoot my own connection for several hours before getting to the point where I could follow along with the course.

## Step 1: Setting Up a MySQL Database on RDS

This is pretty painless but somewhat nerve racking as there are a ton of options and I found myself getting terrified at each step that I was going to make some erroneous and irreversible choice.

### Step 1A: Setting up your account

[This page](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html) walks through 

* setting up an AWS account
* setting up IAM (Identity and Access Management)
* determining/setting up requirements

I DID NOT TAKE ANY ACTION ON THE LAST 2 BULLET POINTS. I'm sure these two things are important if you really want to get cooking with AWS. My experience was that (i) setting up the IAM user account was not crucial to getting a DB up and running and (ii) I did not have to carefully consider my system requirements because I am working with Amazon's "free tier" which means I didn't really have a lot of choices regarding storage.

Note: I did end up having to make some adjustments to my "Security Group" which I will discuss in the next section. I believe that this account set up step would have been a good place to address the security issues that ultimately cost me some time...but I can't really speak to this in an informed manner because it's not the way I did it. [In particular look at the "Provide Access to Your DB Instance in Your VPC by Creating a Security Group" Section here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html#CHAP_SettingUp.IAM)


### Step 1B: Set up a test database

First, I go to my AWS Console and navigate to the RDS service. 

![aws-console](/images/aws-rds-home.png)

Note: you may have to a seach for 'RDS' in order to get to the right AWS service (there are lots of them).

Next, you want 'create database'

![create-db](/images/aws-rds-create-db.png)

This next part is where you have a lot of choices to make. I am looking for the simplest possible path here so I chose a MySQL database and the free tier. Note: by choosing Amazon's "free tier" a lot of decisions get made automagically for you.

![db-setup](/images/db-setup-1.png)

Here are the headers and choices I made for each in setting up my database:

* Engine Type: MySQL
* Version: whatever the default was
* Templates: "free tier"
* Settings:
    - DB cluster identifier: "mams-test"...pretty sure this doesn't matter just pick a name for your instance
    - Master user name: user
    - Master password:  password
* DB Instance Size: since I choose the "free tier" there was nothing to do here
* Storage: I just accepted all the defaults
* Availability and Durability: there was nothing for me to do here
* Connectivity: accepted default
    - Additional connectivity configuration: 
        - Subnet group: accepted default
        - Publicly accessible: "yes"
        - VPC security group: "choose existing"
* Database Authentication: password authentication

After filling in these parameters, and with great hesitation, I hit "create database."

Here is the resulting screen:

![db-complete](/images/db-complete.png)

I assumed that the 'available' flag under status was a good sign. 

## Step 2: Connect to the database 

Here is where the rubber meets the road for me. I'm not a database engineer, I'm an analytics guy. I really only care about databases inasmuch as they are a useful piece of architecture that helps pipe data to my R or Python machines effectively. Generally speaking, what I care most about is being able to get data out of a database and into some analytics software.

This is also the step that tripped me up and resulted in several hours of troubleshooting.

### Step 2A: Get MySQL Workbench

I already had this product installed but if you're playing along at home and don't already have the MySQL Workbench, just [go here](https://dev.mysql.com/downloads/workbench/) (or google "download MySQL Workbench). It's pretty painless...same as installing SQL Developer, TOAD, or any other SQL development environment.

I didn't configure anything, just fired up the MySQL Workbench out-of-the-box.

### Step 2B: Connect MySQL Workbench to the RDS database

Here, we need 1 critical piece of info from the Amazon AWS Console: the database endpoint. We can get this from the database page on our Amazon RDS dashboard:

![db-endpoint](/images/db-endpoint.png)

The way things are supposed to work from here is:

1. we fire up the MySQL Workbench
2. add a new database connection (MySQL Workbench --> Database --> Connect to database)
3. copy the database endpoint to the "Hostname" field, then enter the database username and password that you set up in Amazon RDS

![mysqlworkbench1](/images/sqlworkbench-connect.png)

And there you have it! Easy-peesy-lemon-squeezy, you're connected. If you're lucky. I wasn't. Here's the additional step that I had to conduct to get connected:

### Step 2C: Edit Inbound Security Rules

To do this I had to leave the Amazon RDS Console and navigate back to the AWS Console.

![aws-home](/images/aws-home.png)

Then I navigated to the VPC tab and went into Security Groups. Towards the bottom of the page there is a tab for "Inbound Rules." I clicked to edit my inbound rules to allow all traffic on all port ranges.

![security-group](/images/securitygroup-inbound-rules.png)

Once I edited the inbound rules for my security group, I was able to connect to the cloud database through MySQL Workbench.

![db-connection-good](/images/db-connection.png)

Success!
