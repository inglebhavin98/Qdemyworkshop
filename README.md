# Qdemyworkshop
Created a web application using flask and mysql. Dockerized so that it can be modified to deploy any web content and is easily scalable.

## Introduction  
This directory contains a flask application that POSTs data in json format on the localhost, and a html file to display it. 
Apache server handles displaying the json data on the webpage.
Compose creates two containers :

* Container A : Flask Application
* Container B : apache-php server

##  Installation (Ubuntu)
##  Usage

### Steps to run :
* ```git-clone``` the repository to your local machine / any host OS in a VM running on the cloud. 
* On a terminal type ```docker-compose up``` to get the containers attached and running.
* Use the ```-d``` option for running in detached mode so that you can see both the containers created with ```docker ps``` command.

### From Docker Hub
* pull the containers from my docker hub [repository here](https://cloud.docker.com/u/inglebhavin98).
* To pull server type : ```docker pull inglebhavin98/server```  
* To pull client type : ```docker pull inglebhavin98/client``` 
Run both the containers and check ```localhost:4004``` for server POSTs and ```localhost:4005``` for the webpage.
This demonstrates linking of two services using containers.

## flask-app

This directory contains a web application created using flask and mysql. Dockerized so that it can be modified to deploy any web content and is easily scalable.

It will be a simple bucket list application where users can register, sign in and create their bucket list. We'll be using Flask, a Python web application framework, to create our application, with MySQL as the back end.

> Flask is a microframework for Python based on Werkzeug, Jinja 2 and good intentions.


Container A : Flask Application
Container B : MySQL Server

After the intial setup that is cloning this repo in your machine, run ```app1.py``` to fire up the server and check if everything works fine.
Application is accessed on ```localhost:5002```.

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
