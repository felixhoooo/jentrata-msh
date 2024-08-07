# Jentrata - Message Handler Service

[![Join the chat at https://gitter.im/jentrata/jentrata-msh](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/jentrata/jentrata-msh?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Jentrata MSH is based on Hermes Messaging Gateway developed by CECID, The University of Hong Kong. The goal of Jentrata is build on the original work developed by CECID as well as build a community to continue its development.

See [jentrata.org](http://jentrata.org) for complete details on the project

[Current Build Status](https://jentrata.ci.cloudbees.com/job/jentrata-msh-master/lastBuild)

![Build Status](https://jentrata.ci.cloudbees.com/job/jentrata-msh-master/lastBuild/buildStatus)

## Building/Contributing

### GitHub Setup
1. Fork Repo into your github account
2. Run on your local machine:

		git clone git@github.com:<github-username>/jentrata-msh.git
		cd jentrata-msh.git
		git remote add upstream git@github.com:jentrata/jentrata-msh.git

You can now pull/rebase from the upstream repository

	git pull upstream master
	
And push now push these changes to your forked repository

	git push
	
Also you should run the following

    git config --global user.name "Your Name"
    git config --global user.email <your_email>

### How to Build Jentrata from source
Requires maven 3.0+  and Java 6+
	
	git clone git@github.com:<github-username>/jentrata-msh.git
	cd jentrata-msh
	mvn clean install
	
This will create jentrata-msh-tomcat.tar.gz in jentrata-msh/Dist/target/ as well as an uncompressed version

### How to get Jentrata Running on tomcat using PostgreSQL on unix (including Mac) systems

## Install and configure Tomcat

1. download and install tomcat 6.x or 7.x
2. untar jentrata-msh-tomcat.tar.gz
3. set JENTRATA_HOME and TOMCAT_HOME environment variables, for example

		export JENTRATA_HOME=/dev/jentrata-msh-tomcat
		export TOMCAT_HOME=/dev/tomcat
		
3. create a symlink to the $JENTRATA_HOME/webapps/corvus in the $TOMCAT_HOME/webapps directory to

		ln -s $JENTRATA_HOME/webapps/corvus $TOMCAT_HOME/webapps/jentrata

4. set JAVA_OPTS environment variable as follows

		export JAVA_OPTS="$JAVA_OPTS -Dcorvus.home=$JENTRATA_HOME"

5. add the following roles and user to $TOMCAT_HOME/conf/tomcat-users.xml

		<role rolename="admin"/>
		<role rolename="corvus"/>
		<user username="corvus" password="corvus" roles="tomcat,admin,corvus"/>

## Install and configure database

### PostgreSQL

1. install postgresql 8.3+ - During the install, for the default user 'postgres', make the password 'postgres' (ignore quotes). For Mac OSX you should read the link - [memory configuration info for OSX](http://support.bitrock.com/article/postgresql-cannot-allocate-memory-on-mac-os-x)

2. copy the database creation scripts to your PostgreSQL install

		copy JENTRATA_HOME/sql/ebms.sql to the PostgreSQL bin directory
		copy JENTRATA_HOME/sql/as2.sql to the PostgreSQL bin directory

3. log in as the PostgreSQL user 'postgres' created during the install

4. set the password for the default user 'postgres' as a temporary environment variable

		export PGPASSWORD="postgres"

5. cd to the PostgreSQL bin directory

6. create a postgres user corvus with default password corvus

		./createuser -s -d -P corvus
		
7. create the ebms and as2 databases

		./createdb -O corvus ebms
		./createdb -O corvus as2
		
8. Run the db create tables scripts

		./psql -f ebms.sql ebms
		./psql -f as2.sql as2

9. Logout as user postgres

### MySQL

1. Install MySQL for your OS (http://mysql.com)

2. Create your databases

		mysql -e "CREATE DATABASE ebms;"
		mysql -e "CREATE DATABASE as2;"

3. Create users/permissions to new databases

		mysql -e "GRANT ALL ON ebms.* TO 'ebms'@'localhost' IDENTIFIED BY 'ebms';"
		mysql -e "GRANT ALL ON as2.* TO 'as2'@'localhost' IDENTIFIED BY 'as2';"

4. Import DB Schema

		mysql ebms < JENTRATA_HOME/sql/mysql_ebms.sql
		mysql as2 < JENTRATA_HOME/sql/mysql_as2.sql

5. Configure Jentrata to use MySQL

	You will need to modify the some parameters of the doafactory component in the 'plugins' directory for each plugin of Jentrata that you wish to use with MySQL.
    
		JENTRATA_HOME/plugins/hk.hku.cecid.ebms/conf/hk/hku/cecid/ebms/spa/conf/ebms.module.xml
		JENTRATA_HOME/plugins/hk.hku.cecid.edi.as2/conf/hk/hku/cecid/edi/as2/conf/as2.module.core.xml

	Look for the following Component:
        
		<component id="daofactory" name="System DAO Factory">

	Find the following parameters and change them as follows:

		<parameter name="driver" value="com.mysql.jdbc.Driver"/>
		<parameter name="url" value="jdbc:mysql://127.0.0.1:3306/ebms"/>

## Run Jentrata

1. Start tomcat and browse to [http://localhost:8080/jentrata/admin/home](http://localhost:8080/jentrata/admin/home). You will need to login using the username and password you set in the tomcat-users.xml corvus/corus by default

2. If Jentrata doesn't start correctly you can check the various log files under $TOMCAT_HOME/logs/ or $JENTRATA_HOME/logs for errors

## Running Jentrata with Docker

Jentrata provides a Dockerfile and also a docker-compose.yml file. The Dockerfile is used to buid a general docker
image for Jentrata. The docker-compose file is to speed up development. It contains preset variables.

### Notes

If Jentrata's docker container doesn't detect a postgresql server on the configured hostname and port, it will pause
and retry to connect until one is made available.

### Requirements

Jentrata in docker assumes that you're using Postgresql. The docker-compose sets up a docker image with this preconfigured.

### Start Jentrata using Docker Compose

Docker compose makes it easy to bring up a series of docker images in conjunction with each other.

        docker-compose up

This has everything pre-configured and ready to go. You can acccess Jentrata on: [http://localhost:8080/jentrata/admin/home](http://localhost:8080/jentrata/admin/home)

### Extra: Build the docker container

If you want to build the checked out version of Jentrata, or you want to avoid using the docker hub you can build the
docker image like follows:

        docker build -t jentrata/jentrata-msh .

#### More docker information

See: https://github.com/jentrata/jentrata-msh-docker


#### Notes
#### Notes
docker build -t jentrata/jentrata-msh .

docker run -d --name jentrata-db -v "C:/Users/User/IdeaProjects/jentrata-msh/Dist/src/main/scripts/sql/as2.sql:/work/sql/as2.sql" -v "C:/Users/User/IdeaProjects/jentrata-msh/Dist/src/main/scripts/sql/ebms.sql:/work/sql/ebms.sql" -v "C:/Users/User/IdeaProjects/jentrata-msh/ContainerFiles/initdb.sh:/docker-entrypoint-initdb.d/initdb.sh" -e "POSTGRES_USER=jentrata" -e "POSTGRES_PASSWORD=jentrata" -e "POSTGRES_DB=jentrata" -e "DB_USER_NAME=corvus" -e "DB_USER_PASS=corvus" postgres:9.5

docker run -d --name jentrata --link jentrata-db:db -p 8080:8080 jentrata/jentrata-msh

docker run -d --name jentrata-db2 -v "C:/Users/User/IdeaProjects/jentrata-msh/Dist/src/main/scripts/sql/as2.sql:/work/sql/as2.sql" -v "C:/Users/User/IdeaProjects/jentrata-msh/Dist/src/main/scripts/sql/ebms.sql:/work/sql/ebms.sql" -v "C:/Users/User/IdeaProjects/jentrata-msh/ContainerFiles/initdb.sh:/docker-entrypoint-initdb.d/initdb.sh" -e "POSTGRES_USER=jentrata" -e "POSTGRES_PASSWORD=jentrata" -e "POSTGRES_DB=jentrata" -e "DB_USER_NAME=corvus" -e "DB_USER_PASS=corvus" postgres:9.5

docker run -d --name jentrata2 --link jentrata-db2:db -p 8081:8080 jentrata/jentrata-msh

docker exec -it jentrata bash 
docker exec -it jentrata2 bash

http://localhost:8080/jentrata/admin/home
http://localhost:8081/jentrata/admin/home

1. Copy files inside jentrata-msh-3.x-SNAPSHOT-tomcat.tar to c:\jentrata
2. For Error: Could not find or load main class hk.hku.cecid.corvus.http.EBMSPartnershipSender, copy jentrata-corvus-wsclient-3.x-SNAPSHOT to C:\jentrata\sample\lib
3. Copy testpayload to C:\jentrata\sample\config\ebms-send
4. Run script ebms-partnership to create partnership
5. Run script ebms-send, this will


kubectl port-forward -n kong service/kong-cp-kong-manager 8002 --address 0.0.0.0
kubectl port-forward -n kong service/kong-dp-kong-proxy 80 --address 0.0.0.0
kubectl port-forward -n kong service/kong-cp-kong-admin 8001 --address 0.0.0.0

kubectl port-forward -n redis service/my-release-redis-master 6379 --address 0.0.0.0

kubectl port-forward -n redis service/my-release-redis-cluster 6379 --address 0.0.0.0

redis password:

kubectl port-forward -n redis-sentinel service/redis-sentinel-headless 26379 --address 0.0.0.0

redis password:


kubectl exec -i -t dnsutils -- nslookup redis-master-0.redis-headless.default.svc.cluster.local


helm upgrade kong-cp kong/kong -n kong --values ./values-cp.yaml
helm upgrade kong-dp kong/kong -n kong --values ./values-dp.yaml



ebms=# select message_id, message_box, message_type, action, time_stamp, status, status_description from message order by time_stamp asc;
message_id            | message_box |  message_type   |     action     |       time_stamp        | status |       status_description       
----------------------------------+-------------+-----------------+----------------+-------------------------+--------+--------------------------------
20240515-001608-72900@172.17.0.4 | outbox      | Order           | action         | 2024-05-15 00:16:08.731 | PS     | Acknowledgement is received
20240515-001608-72900@172.17.0.4 | inbox       | Order           | action         | 2024-05-15 00:16:08.731 | DL     | Message is delivered
20240515-001608-89301@172.17.0.4 | outbox      | Acknowledgement | Acknowledgment | 2024-05-15 00:16:08.893 | PS     | Message was sent synchronously
20240515-001608-89301@172.17.0.4 | inbox       | Acknowledgement | Acknowledgment | 2024-05-15 00:16:08.893 | PS     | Message is processed


keytool -genkey -alias cecid -keyalg RSA -keystore cecid.p12 -storetype pkcs12 -storepass 000000 -keypass 000000 -sigalg SHA1withRSA

keytool -exportcert -alias cecid -keystore cecid.p12 -storetype pkcs12 -file cecid.cer


docker run --name redis-host -d redis -p 6376:6379
docker run --network=kong-net --name redis-host -d redis
docker exec -it redis-host redis-cli