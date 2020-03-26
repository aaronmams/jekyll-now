I've been wanting to beef-up my AWS skills for a long time. The main thing that's slowed me down is that we cannot store government data on/in Amazon's AWS ecosystem. This isn't really a hard roadblock, it's just that a lot of my blog content is generated from little programming or data hurdles I encounter at work. 

Well, I picked up a little side project and decided it would be fun to finally try and set up a database on AWS. I suspect this side project will end up generating a few blog posts. This one is dedicated to what should be the relatively simple, discrete task of:

1. setting up a relational database on AWS
2. connecting to that database using MySQL Workbench.

## Goal

The goal in this post is a simple proof-of-concept: get a relational database set up in Amazon's RDS Service and access that database from my laptop using some open source (off-the-shelf) database management software.

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

![aws-console]('/images/aws-rds-home.png')

Note: you may have to a seach for 'RDS' in order to get to the right AWS service (there are lots of them).

Next, you want 'create database'

![create-db]('images/aws-rds-create-db.png')

This next part is where you have a lot of choices to make. I am looking for the simplest possible path here so I chose a MySQL database and the free tier. Note: by choosing Amazon's "free tier" a lot of decisions get made automagically for you.

![db-setup]('images/db-setup-1.png')




