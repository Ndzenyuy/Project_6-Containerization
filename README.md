# Project 6: Containerising a webapp
In Projects 1 & 2, our webapp was locally set up step by step on a local machine. This was very necessary for the present project since to containerize a webapp, a prior mastery of the different setup steps is required. In this project, the configuration steps were rather containerized using Docker. I used base images(openjdk, tomcat, nginx) to configure and build the images for each tier(three tiers) to work according to our architecture. To reduce the image size, I used a multistage dockerfile to build and run our artifact, since building artifact needed so many downloaded dependencies. In order to build and test the different images simultaneously, docker-compose was very helpful. Containerizing microservices solves the problems of resource wastage, human errors during deployments. It just deploys on as much resources as needed. The containers were then run on one t2.small EC2 instance.

## Project architecture
![](https://github.com/Ndzenyuy/project6_containerization/blob/Main/images/Containerisation.jpg)
## Technologies
- MySQL
- Docker
- Tomcat
- Nginx
- Memcached
- RabbitMQ

## Prerequisites
- Dockerhub account
- AWS account

## Steps
1. Setup Docker Engine
   Launch an ec2 instance with the following configurations
   ```
      OS: ubuntu 22.04
      keypair: <creat_a_keypair>
      sec. group inbound rule: Allows all trafic from my IP
   ```
   ssh into the ec2, t2.small instance, then install docker engine with the following commands(verify official installation steps for updated commands):
```
   sudo apt-get update
   sudo apt-get install ca-certificates curl gnupg -y
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg

   echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

```
Add ubunter user into docker group by running the following command
```
   usermod aG docker ubuntu
```

3. Dockerhub setup
   - Get the source code, fork the github repo to your own repository, then clone it to our local system
   ```
      https://github.com/Ndzenyuy/vprofile-project.git
   ```
   - Open the project folder with VScode and Switch to the branch container by running in terminal
   ```
   git checkout containers
   ```
   - Create repositories with the names of our images to be pushed
     * <your_dockerhub_user_name>/vprofileweb
     * <your_dockerhub_user_name>/vprofiledb
     * <your_dockerhub_user_name>/vprofileapp
     
4. App image Dockerfile
   Create a folder _docker-files_, create a sub folder _app_ and in it place Dockerfile then insert the following
```
 FROM openjdk:11 AS BUILD_IMAGE
 RUN apt update && apt install maven -y
 RUN git clone https://github.com/ndzenyuy/vprofile-project.git
 RUN cd vprofile-project && git checkout docker && mvn install

 FROM tomcat:9-jre11
 LABEL "Project"="Vprofile"
 LABEL "Author"="Jones"
 RUN rm -rf /usr/local/tomcat/webapps/*
 COPY --from=BUILD_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

 EXPOSE 8080
 CMD ["catalina.sh","run"]

```
5. DB image Dockerfile
   In the folder _docker-files_ create a subfolder db. create two files Dockerfile and db_backup.sql
   In the Dockerfile, paste the following:
```
FROM mysql:8.0.33
LABEL "Project"="Vprofile"
LABEL "Author"="Jones"

ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"

ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql
```

In db_backup.sql
```
-- MySQL dump 10.13  Distrib 5.7.18, for Linux (x86_64)
--
-- Host: localhost    Database: accounts
-- ------------------------------------------------------
-- Server version	5.7.18-0ubuntu0.16.10.1

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `role`
--

DROP TABLE IF EXISTS `role`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `role`
--

LOCK TABLES `role` WRITE;
/*!40000 ALTER TABLE `role` DISABLE KEYS */;
INSERT INTO `role` VALUES (1,'ROLE_USER');
/*!40000 ALTER TABLE `role` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `user`
--

DROP TABLE IF EXISTS `user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `userEmail` varchar(255) DEFAULT NULL,
  `profileImg` varchar(255) DEFAULT NULL,
  `profileImgPath` varchar(255) DEFAULT NULL,
  `dateOfBirth` varchar(255) DEFAULT NULL,
  `fatherName` varchar(255) DEFAULT NULL,
  `motherName` varchar(255) DEFAULT NULL,
  `gender` varchar(255) DEFAULT NULL,
  `maritalStatus` varchar(255) DEFAULT NULL,
  `permanentAddress` varchar(255) DEFAULT NULL,
  `tempAddress` varchar(255) DEFAULT NULL,
  `primaryOccupation` varchar(255) DEFAULT NULL,
  `secondaryOccupation` varchar(255) DEFAULT NULL,
  `skills` varchar(255) DEFAULT NULL,
  `phoneNumber` varchar(255) DEFAULT NULL,
  `secondaryPhoneNumber` varchar(255) DEFAULT NULL,
  `nationality` varchar(255) DEFAULT NULL,
  `language` varchar(255) DEFAULT NULL,
  `workingExperience` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES (7,'admin_vp','admin@visualpathit.com',NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,'$2a$11$0a7VdTr4rfCQqtsvpng6GuJnzUmQ7gZiHXgzGPgm5hkRa3avXgBLK'),(8,'WahidKhan','wahid.khan74@gmail.com',NULL,NULL,'28/03/1994','M Khan','R Khan','male','unMarried','Ameerpet,Hyderabad','Ameerpet,Hyderabad','Software Engineer','Software Engineer','Java HTML CSS ','8888888888','8888888888','Indian','english','2 ','$2a$11$UgG9TkHcgl02LxlqxRHYhOf7Xv4CxFmFEgS0FpUdk42OeslI.6JAW'),(9,'Gayatri','gayatri@gmail.com',NULL,NULL,'20/06/1993','K','L','male','unMarried','Ameerpet,Hyderabad','Ameerpet,Hyderabad','Software Engineer','Software Engineer','Java HTML CSS ','9999999999','9999999999','India','english','5','$2a$11$gwvsvUrFU.YirMM1Yb7NweFudLUM91AzH5BDFnhkNzfzpjG.FplYO'),(10,'WahidKhan2','wahid.khan741@gmail.com',NULL,NULL,'28/03/1994','M Khan','R Khan','male','unMarried','Ameerpet,Hyderabad','Ameerpet,Hyderabad','Software Engineer','Software Engineer','Java HTML CSS ','7777777777','777777777','India','english','7','$2a$11$6oZEgfGGQAH23EaXLVZ2WOSKxcEJFnBSw2N2aghab0s2kcxSQwjhC'),(11,'KiranKumar','kiran@gmail.com',NULL,NULL,'8/12/1993','K K','RK','male','unMarried','California','James Street','Software Engineer','Software Engineer','Java HTML CSS ','1010101010','1010101010','India','english','10','$2a$11$EXwpna1MlFFlKW5ut1iVi.AoeIulkPPmcOHFO8pOoQt1IYU9COU0m'),(12,'Saikumar','sai@gmail.com',NULL,NULL,'20/06/1993','Sai RK','Sai AK','male','unMarried','California','US','Software Engineer','Software Engineer','Java HTML CSS AWS','8888888111','8888888111','India','english','8','$2a$11$pzWNzzR.HUkHzz2zhAgqOeCl0WaTgY33NxxJ7n0l.rnEqjB9JO7vy'),(13,'RamSai','ram@gmail.com',NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,'$2a$11$6BSmYPrT8I8b9yHmx.uTRu/QxnQM2vhZYQa8mR33aReWA4WFihyGK');
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `user_role`
--

DROP TABLE IF EXISTS `user_role`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `user_role` (
  `user_id` int(11) NOT NULL,
  `role_id` int(11) NOT NULL,
  PRIMARY KEY (`user_id`,`role_id`),
  KEY `fk_user_role_roleid_idx` (`role_id`),
  CONSTRAINT `fk_user_role_roleid` FOREIGN KEY (`role_id`) REFERENCES `role` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `fk_user_role_userid` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `user_role`
--

LOCK TABLES `user_role` WRITE;
/*!40000 ALTER TABLE `user_role` DISABLE KEYS */;
INSERT INTO `user_role` VALUES (4,1),(5,1),(6,1),(7,1),(8,1),(9,1),(10,1),(11,1),(12,1),(13,1);
/*!40000 ALTER TABLE `user_role` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2017-12-07 16:32:31

```

6. Web image Dockerfile
Create a third subfolder under docker-files named web. In it we have
Dockerfile:
```
FROM nginx
LABEL "Project"="Vprofile"
LABEL "Author"="Jones"

RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginxvproapp.conf /etc/nginx/conf.d/vproapp.conf
```
nginxvproapp.conf:
```
upstream vproapp {
    server vproapp:8080;
}
server {
    listen 80;
    location / {
        proxy_pass http://vproapp;
    }
}
```

7. Docker compose
   In order to build all three images together, we use docker-compose.yml. In it we paste the code
```
version: '3.8'
services:
  vprodb:
    build: 
      context: ./Docker-files/db
    image: ndzenyuy/vprofiledb
    container_name: vprodb
    ports:
      - "3306:3306"
    volumes:
      - vprodbdata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=vprodbpass

  vprocache01:
    image: memcached
    ports:
      - "11211:11211"

  vpromq01:
    image: rabbitmq
    ports:
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest

  vproapp:
    build: 
      context: ./Docker-files/app
    image: ndzenyuy/vprofileapp
    container_name: vproapp
    ports:
      - "8080:8080"
    volumes:
      - vproappdata:/usr/local/tomcat/webapps
  
  vproweb:
    build: 
      context: ./Docker-files/web
    image: ndzenyuy/vprofileweb
    container_name: vproweb
    ports:
      - "80:80"

volumes:
  vprodbdata: {}
  vproappdata: {}

```
We then save and push this code to our GitHub repository.

8. Build and run
   SSH into our instance and clone the project to it. cd into vprofile-project folder and run
```
docker compose build 
```
To verify the build images, we can run
```
docker compose images
```
To run our application through containers, we run
```
docker compose up -d
```
To check the running webapp, copy the public IP of the ec2 instance and run it on the browser.

9. Clean up
    ```
    docker compose down
    ```
    Delete all the images from the ec2 instance and shut it down
   
## Results
![](https://github.com/Ndzenyuy/project6_containerization/blob/Main/images/Screenshot%20from%202023-08-17%2014-20-55.png)
![](https://github.com/Ndzenyuy/project6_containerization/blob/Main/images/Screenshot%20from%202023-08-17%2014-21-34.png)

