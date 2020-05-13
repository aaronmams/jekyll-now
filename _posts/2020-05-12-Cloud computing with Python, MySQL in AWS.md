Another post I'm putting up mostly so I have something to refer to in a week when I forget how I did this.

**What I did**: Established a connection between the python interpreter running on my AWS EC2 Ubuntu server and the MySQL database instance in AWS RDS.

Here are the steps:
1. installed the mysql.connector module on the EC2 server (this should have been trivial but sadly was not)
2. connected to the mintyhippo database in RDS using the mysql.connector.connect() method
3. confirmed the connection by printing out rows of a database table

## Step 1: Import the mysql.connector Module

As a first pass the really simple pip install seemed to work fine for me:

```bash
ubuntu@ip-192-168-2-247:~$ pip install mysql-connector
Collecting mysql-connector
  Downloading https://files.pythonhosted.org/packages/28/04/e40098f3730e75bbe36a338926f566ea803550a34fb50535499f4fc4787a/mysql-connector-2.2.9.tar.gz (11.9MB)
    100% |████████████████████████████████| 11.9MB 97kB/s 
Building wheels for collected packages: mysql-connector
  Running setup.py bdist_wheel for mysql-connector ... done
  Stored in directory: /home/ubuntu/.cache/pip/wheels/8c/83/a1/f8b6d4bb1bd6208bbde1608bbfa7557504bed9eaf2ecf8c175
Successfully built mysql-connector
Installing collected packages: mysql-connector
Successfully installed mysql-connector
You are using pip version 8.1.1, however version 20.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command. 
```

However, I wanted to check to make sure the module was available so I did

```bash
ubuntu@ip-192-168-2-247:~$ python3
[>>> help('modules')

```

and was dismayed to see that the mysql connector module was not displayed.

I discovered however that if I displayed the module list from the python 2 version on the server

```bash
ubuntu@ip-192-168-2-247:~$ python2
[>>> help('modules')
```

the module was there.

So issue #1 (still unresolved) is how do I get the pip install process to make libraries available in my Python 3 environment...not the Python 2 environment?

## Step 2: Connect to the Database

Step 2 was pretty textbook. I lifted the sample connection string right off the [MySQL Dev Page](https://dev.mysql.com/doc/connector-python/en/connector-python-example-connecting.html):

```bash
ubuntu@ip-192-168-2-247:~$ python2
>>> import mysql.connector
>>> cxn = mysql.connector.connect(user='******',password='*********',host='mams-california.c65i4tmttvql.us-west-1.rds.amazonaws.com',database='mintyhippo')
```

## Step 3: Test the Connection

To test the connection I'm going to print out rows of a database table. So I'll start by looking at the database table to see what result I should expect if everything worked properly. [In this post](https://aaronmams.github.io/AWS-MySQL-DB-in-a-Custom-VPC/) I demonstrated how to use mysql on the EC2 Instance to explore the RDS database from the Mac Terminal. 

```bash
ubuntu@ip-192-168-2-247:~$ mysql -h mams-california.c65i4tmttvql.us-west-1.rds.amazonaws.com -u ****** -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11610
Server version: 5.7.26-log Source distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;
ERROR 1046 (3D000): No database selected
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| innodb             |
| mintyhippo         |
| mysql              |
| performance_schema |
| sys                |
| tmp                |
+--------------------+
7 rows in set (0.00 sec)

mysql> use mintyhippo;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------------+
| Tables_in_mintyhippo |
+----------------------+
| USERS                |
+----------------------+
1 row in set (0.00 sec)

mysql> select * from users;
ERROR 1146 (42S02): Table 'mintyhippo.users' doesn't exist
mysql> select * from USERS;
+----+-----------+----------+-----------------------+
| Id | Firstname | Lastname | Email                 |
+----+-----------+----------+-----------------------+
|  1 | aaron     | mamula   | aaron.mamula@noaa.gov |
|  2 | aaron     | mamula   | aaron.mams@gmail.com  |
|  3 | aaron     | mamula   | aamamula@ucsc.edu     |
+----+-----------+----------+-----------------------+
3 rows in set (0.00 sec)
```

So when I connect my Python Instance to the RDS Database, I'll know I've done it right if I can replicate the result above. 

```bash
ubuntu@ip-192-168-2-247:~$ python2
Python 2.7.12 (default, Apr 15 2020, 17:07:12) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.

>>> import mysql.connector
>>> cxn = mysql.connector.connect(user='****',password='*******',host='mams-california.c65i4tmttvql.us-west-1.rds.amazonaws.com',database='mintyhippo')
>>> cursor=cxn.cursor()
>>> cursor.execute("SELECT * FROM USERS")
>>> row = cursor.fetchone()
>>> while row is not None:
...     print(row)
...     row=cursor.fetchone()
... 
(1, u'aaron', u'mamula', u'aaron.mamula@noaa.gov')
(2, u'aaron', u'mamula', u'aaron.mams@gmail.com')
(3, u'aaron', u'mamula', u'aamamula@ucsc.edu')

``` 

It's a small victory but an important one for me. It's a proof of concept. If I can connect the Python version running on the EC2 server to the RDS Database using command line tools then I can probably figure out how to do it in a .py file. That means I can write larger data analytics scripts that pull from the MySQL database on RDS and I can run the script on the EC2 Ubuntu server (probably). 


