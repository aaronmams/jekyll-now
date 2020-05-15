<img src "/images/fetchone-vim.png" width=400 height=200 />

[In my last post](https://aaronmams.github.io/Cloud-computing-with-Python,-MySQL-in-AWS/) I established a python connection to a MySQL database running on in AWS/RDS. In this follow-up I want to do essentially the same thing but I'd like to do it through a python script. 

There are 3 parts to this:

1. write the python scripts that will connect to the database
2. upload those scripts to my cloud-based Ubuntu server running in AWS/EC2
3. run the scripts

## Step 1: Write the python scripts

This step is somewhat trivial as I'm just using some recycled open source code. Working from the outside in I have:

1. fetchone.py: a python script that connects to a database and prints out each row in one of the database tables

2. python_mysql_dbconfig.py: a script that contains a user-defined function for connecting to a MySQL database

3. config.ini: the function in python_mysql_dbconfig.py will read configuration parameters from a file called config.ini. 

Here is the fetchone.py file

```python
#!/usr/bin/python

from mysql.connector import MySQLConnection, Error
from python_mysql_dbconfig import read_db_config

def query_with_fetchone():
    try:
        dbconfig = read_db_config()
        conn = MySQLConnection(**dbconfig)
        cursor = conn.cursor()
        cursor.execute("Select * from USERS")
        
        row = cursor.fetchone()
        
        while row is not None:
            print(row)
            row = cursor.fetchone()
        
    except Error as e:
        print(e)
        
    finally:
        cursor.close()
        conn.close()

if __name__ == '__main__':
    query_with_fetchone() 
```

You can see that fetchone.py imports the method read_db_config from the module python_mysql_dbconfig. Here is the python_mysql_dbconfig.py file that contains the needed method. This module was lifted more-or-less verbatim [from this MySQL Tutorial](https://www.mysqltutorial.org/python-connecting-mysql-databases/).

```python

from configparser import ConfigParser


def read_db_config(filename='config.ini', section='mysql'):
    """ Read database configuration file and return a dictionary object
    :param filename: name of the configuration file
    :param section: section of database configuration
    :return: a dictionary of database parameters
    """
    # create parser and read ini configuration file
    parser = ConfigParser()
    parser.read(filename)

    # get section, default to mysql
    db = {}
    if parser.has_section(section):
        items = parser.items(section)
        for item in items:
            db[item[0]] = item[1]
    else:
        raise Exception('{0} not found in the {1} file'.format(section, filename))

    return db
```

You can also see that python_mysql_dbconfig is looking for a file called 'config.ini'. So, finally, here is the config.ini file

```python
[mysql]
host= mams-california.c65i4tmttvql.us-west-1.rds.amazonaws.com
database= mintyhippo
user = ******
password = *******
```

## Step 2: Send these files to the AWS server

I have created a directory in my AWS Ubuntu server called rdstest. This is where I will be storing the aforementioned files.

Moving .py files to the AWS server. The first argument is the key-pair file allowing a connection to the remote server, the second argument is the file I want to me to the sever, the third argument is the dns or server endpoint.

```bash
(base) aarons-MacBook-Air-2:~ aaronmamula$ scp -i ~/Desktop/*****.pem ~/Desktop/AWSpython/python_mysql_dbconfig.py ubuntu@ec2-54-153-106-13.us-west-1.compute.amazonaws.com:~/rdstest
```

After the file transfer I log back into my AWS server and check that the files I want are where I want them:

```bash
(base) aarons-MacBook-Air-2:~ aaronmamula$ cd ~/Desktop
(base) aarons-MacBook-Air-2:Desktop aaronmamula$ ssh -i "*******.pem" ubuntu@ec2-54-153-106-13.us-west-1.compute.amazonaws.com
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1101-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Ubuntu 20.04 LTS is out, raising the bar on performance, security,
   and optimisation for Intel, AMD, Nvidia, ARM64 and Z15 as well as
   AWS, Azure and Google Cloud.

     https://ubuntu.com/blog/ubuntu-20-04-lts-arrives

28 packages can be updated.
7 updates are security updates.

*** System restart required ***
Last login: Thu May 14 03:51:58 2020 from 67.180.160.152
ubuntu@ip-192-168-2-247:~$
ubuntu@ip-192-168-2-247:~/rdstest$ ls -al
total 24
drwxrwxr-x 2 ubuntu ubuntu 4096 May 14 03:27 .
drwxr-xr-x 6 ubuntu ubuntu 4096 May 13 04:59 ..
-rw-r--r-- 1 ubuntu ubuntu  129 Apr 23 04:51 config.ini
-rwxr-xr-x 1 ubuntu ubuntu  622 Apr 24 15:18 fetchone.py
-rw-r--r-- 1 ubuntu ubuntu   19 Apr 12 01:35 hello_aws.py
-rw-r--r-- 1 ubuntu ubuntu  732 May 14 03:27 python_mysql_dbconfig.py
```
## Step 3: Run the fetchone.py script

### Step 3A: Install the configparser library

The python scripts outlined in Step 2 above have two external library dependencies: [mysql-connector](https://pypi.org/project/mysql-connector-python/) and [configparser](https://pypi.org/project/configparser/). I previously installed the mysql-connector module on the AWS server so that should be ready to go. I don't believe I have the configparser library so I will need to pip install that.

### Step 3B: Test the read_db_config() method

Test my python_mysql_dbconfig module. The read_db_config() method in python_mysql_dbconfig.py is pretty simple. It reads database parameters from a file called 'config.ini' and, as long as that file has a 'mysql' section, the method returns the configuration parameters from the config.ini file. So, running this through the python interpreter on my EC2 server, I should just get back a list of connection parameters for my RDS MySQL database: 

```bash
ubuntu@ip-192-168-2-247:~$ cd ~/rdstest
ubuntu@ip-192-168-2-247:~/rdstest$ python2
Python 2.7.12 (default, Apr 15 2020, 17:07:12) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from python_mysql_dbconfig import read_db_config
>>> read_db_config()
{u'host': u'mams-california.c65i4tmttvql.us-west-1.rds.amazonaws.com', u'password': u'*********', u'user': u'*****', u'database': u'mintyhippo'}
>>> 

```
### Step 3C: Run the fetchone.py script

I'm going to throw in a little bonus step here just because. I'm going to inspect the fetchone.py script just to double-double confirm that it's what I want to run. I open the file with the VIM text editor that comes pre-installed on the Ubuntu 16.04 AMI that I'm working with.

```bash
ubuntu@ip-192-168-2-247:~$ cd ~/rdstest
ubuntu@ip-192-168-2-247:~/rdstest$ vim fetchone.py

``` 

![fetchone-file](/images/fetchone-vim.png)

Upon inspection everything looks cool so finally, I'll execute the fetchone.py file and, if everything is cool, I should get each row of the USERS tables from the mintyhippo database printed to the screen.

```bash
ubuntu@ip-192-168-2-247:~/rdstest$ python2 fetchone.py
(1, u'aaron', u'mamula', u'aaron.mamula@noaa.gov')
(2, u'aaron', u'mamula', u'aaron.mams@gmail.com')
(3, u'aaron', u'mamula', u'aamamula@ucsc.edu')

```

Yatzee!
