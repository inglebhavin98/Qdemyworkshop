# Qdemyworkshop
Q-Demy workshop organised by Quantiphi at Sardar Patel Institute of Technology on Cloud Technologies.

## Introduction
Students were assigned the task to create an application and dockerize it in a way where two or more containers are created and they communicate with each other when run.

This repository is divided into two parts:

## 1. Simple json Parser

Created a flask application that POSTs data in json format on the localhost, and a web page is used to display it.
Apache server handles the task of running the web-page.
This application was divided in two parts :
* Container A : Flask Application
* Container B : apache-php server

###  Installation (Ubuntu)

1. Git clone this repository using ```git clone <repository_url>```.
2. Open terminal and navigate to ```docker``` directory.
3. Install ```docker``` from [here]().
4. Install ```docker-compose``` from [here]().
5. Check if they are running properly.
6. Run ```sudo docker-compose up -d``` to run the containers in detached mode.
7. Check if they are running by : ```docker ps```.

###  Usage

1.  Go to ```http://localhost:4004```.
2.  This is the raw json string as received by the browser.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/2cont-2.png)
3.  This is the parsed json output on the web page.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/2cont-3.png)
4. These are the two containers that we created. (highlighted in the last lines of output)
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/2cont-5.png)

## 2. Blogging Application 

Created a web application using flask and mysql. Dockerized so that it can be modified to deploy any web content and is easily scalable.

It will be a simple bucket list application where users can register, sign in and create their bucket list, using Flask to create our application, with MySQL as the back end.

> Flask is a microframework for Python based on Werkzeug, Jinja 2 and good intentions.

Container A : Flask Application
Container B : MySQL Server

After the intial setup that is cloning this repo in your machine, run ```app1.py``` to fire up the server and check if everything works fine.
Application is accessed on ```localhost:5002```.



##  Installation (Ubuntu)
1. Git clone this repository using ```git clone <repository_url>```.
2. Open terminal and navigate to ```flask-app``` directory.
3. Install ```docker``` from [here]().
4. Install ```docker-compose``` from [here]().
5. Check if they are running properly.
6. Run ```sudo docker-compose up -d``` to run the containers in detached mode.
7. Check if they are running by : ```docker ps```.

##  Usage
1.  Go to ```http://localhost:5002```.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/1cont-1.png)
2.  Click on signup, enter details and submit.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/1cont-4.png)
Database is updated!!
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/1cont-5.png)
3.  From the main page, click on login and enter credentials.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/login.png)
4.  This is the home page where posts can be viewed.
![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/home.png)
5.  This is the ```AddWish``` page.
 ![alt text](https://github.com/inglebhavin98/Qdemyworkshop/blob/master/images/addwish.png)
 
 
For now you manually need to create the database, tables and Stored procedures. I'll automate that as soon as possible.

##  Setup MySQL 
First, create a database called BucketList. From the command line: 

```

mysql -u <username> -p
  
```
Once the database has been created, create  a table called tbl_user as shown:

```

CREATE TABLE `BucketList`.`tbl_user` (
  `user_id` BIGINT NULL AUTO_INCREMENT,
  `user_name` VARCHAR(45) NULL,
  `user_username` VARCHAR(45) NULL,
  `user_password` VARCHAR(45) NULL,
  PRIMARY KEY (`user_id`));
```

We'll be using Stored procedures for our Python application to interact with the MySQL database. So, once the table tbl_user has been created, create a stored procedure called sp_createUser to sign up a user.
```

DELIMITER $$
CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_createUser`(
    IN p_name VARCHAR(20),
    IN p_username VARCHAR(20),
    IN p_password VARCHAR(20)
)
BEGIN
    if ( select exists (select 1 from tbl_user where user_username = p_username) ) THEN
     
        select 'Username Exists !!';
     
    ELSE
     
        insert into tbl_user
        (
            user_name,
            user_username,
            user_password
        )
        values
        (
            p_name,
            p_username,
            p_password
        );
     
    END IF;
END$$
DELIMITER ;
```
Along with that include the following MySQL configurations:


```
mysql = MySQL()
 
# MySQL configurations
app.config['MYSQL_DATABASE_USER'] = 'jay'
app.config['MYSQL_DATABASE_PASSWORD'] = 'jay'
app.config['MYSQL_DATABASE_DB'] = 'BucketList'
app.config['MYSQL_DATABASE_HOST'] = 'localhost'
mysql.init_app(app)
```

##  Implementing Sign-In
### Creating a Stored Procedure 
To validate a user, we'll need a MySQL stored procedure. So create a MySQL stored procedure as shown:

```DELIMITER $$
CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_validateLogin`(
IN p_username VARCHAR(20)
)
BEGIN
    select * from tbl_user where user_username = p_username;
END$$
DELIMITER ;
```
##  Add Bucket List Items
### Database Implementation
To add items to the bucket list, we need to create a table called tbl_wish.

```
CREATE TABLE `tbl_wish` (
  `wish_id` int(11) NOT NULL AUTO_INCREMENT,
  `wish_title` varchar(45) DEFAULT NULL,
  `wish_description` varchar(5000) DEFAULT NULL,
  `wish_user_id` int(11) DEFAULT NULL,
  `wish_date` datetime DEFAULT NULL,
  PRIMARY KEY (`wish_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=latin1;
```

tbl_wish will have title, description and the ID of the user who created the wish.

Next, we need to create a MySQL stored procedure to add items to the tbl_wish table.
```
USE `BucketList`;
DROP procedure IF EXISTS `BucketList`.`sp_addWish`;
 
DELIMITER $$
USE `BucketList`$$
CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_addWish`(
    IN p_title varchar(45),
    IN p_description varchar(1000),
    IN p_user_id bigint
)
BEGIN
    insert into tbl_wish(
        wish_title,
        wish_description,
        wish_user_id,
        wish_date
    )
    values
    (
        p_title,
        p_description,
        p_user_id,
        NOW()
    );
END$$
 
DELIMITER ;
;

```
##  Display a Bucket List Item
### Stored Procedure to Retrieve a Wish
```
USE `BucketList`;
DROP procedure IF EXISTS `sp_GetWishByUser`;
 
DELIMITER $$
USE `BucketList`$$
CREATE PROCEDURE `sp_GetWishByUser` (
IN p_user_id bigint
)
BEGIN
    select * from tbl_wish where wish_user_id = p_user_id;
END$$
 
```


After database setup is complete you can check by creating a user and logging in ..  create some posts on the wishlist page and all these changes will be reflected in the database.

