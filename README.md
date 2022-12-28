# liquibaseDocumentation
liquibase Documentation


To dockerize Liquibase with MySQL, you will need to create a Dockerfile that includes the necessary steps to set up both Liquibase and MySQL in a Docker container. Here is an example of how you could do this:

Start by specifying a base image that includes the version of MySQL that you want to use. For example:

FROM mysql:8.0

Next, you will need to install Liquibase. You can do this by adding the following commands to your Dockerfile:

RUN apt-get update && apt-get install -y wget unzip
RUN wget https://github.com/liquibase/liquibase/releases/download/v4.2.0/liquibase-4.2.0.zip
RUN unzip liquibase-4.2.0.zip


Next, you will need to create a script that will initialize the MySQL database and run Liquibase to apply your database changes. You can do this by creating a file called "init-db.sh" with the following contents:

#!/bin/bash

# Start MySQL server
mysqld &

# Wait for MySQL to start
sleep 10

# Create the database
mysql -u root -e "CREATE DATABASE mydatabase;"

# Run Liquibase to apply the database changes
liquibase --changeLogFile=/changes/changelog.xml --url="jdbc:mysql://localhost:3306/mydatabase" --username=root --password=password update

Add the "init-db.sh" script to your Docker image by adding the following line to your Dockerfile:

COPY init-db.sh /docker-entrypoint-initdb.d/

Finally, add the following line to your Dockerfile to specify the command that should be run when the container starts:

CMD ["mysqld"]

To build the Docker image, run the following command in the directory containing your Dockerfile:
docker build -t myimage .

To run the Docker container, use the following command:
docker run -p 3306:3306 myimage

This will start the MySQL server and run Liquibase to apply your database changes. You should now be able to connect to the MySQL server using a MySQL client and see the changes that were made by Liquibase.

========================================
Second option

Create a Dockerfile for your Liquibase project.

In the Dockerfile, specify the base image that you want to use for your Liquibase container. For example, you might use the mysql:5.7 image as the base image.

In the Dockerfile, copy the Liquibase jar file and your Liquibase changelog file into the container.

In the Dockerfile, define a command that runs Liquibase using the jar file and changelog file that you just copied into the container.

Build the Docker image using the Dockerfile.

Run a container from the image, making sure to specify the necessary environment variables and volumes.

Here's an example Dockerfile that demonstrates how to dockerize Liquibase with MySQL:

FROM mysql:5.7

COPY liquibase.jar /usr/local/liquibase/
COPY changelog.xml /usr/local/liquibase/

CMD java -jar /usr/local/liquibase/liquibase.jar \
         --driver=com.mysql.jdbc.Driver \
         --classpath=/usr/local/liquibase/liquibase.jar:/usr/share/java/mysql-connector-java.jar \
         --changeLogFile=/usr/local/liquibase/changelog.xml \
         --url="jdbc:mysql://mysql:3306/mydatabase" \
         --username=myuser \
         --password=mypass \
         update

To build the image, you can use the following command:
docker build -t my-liquibase .

To run a container from the image, you can use the following command:
docker run -e MYSQL_ROOT_PASSWORD=mypass -e MYSQL_DATABASE=mydatabase -e MYSQL_USER=myuser -e MYSQL_PASSWORD=mypass -v /my/local/liquibase/changelog.xml:/usr/local/liquibase/changelog.xml my-liquibase

This will start a container with MySQL and Liquibase, and run the Liquibase update command to apply the changes specified in the changelog file to the database.


=======================================
Third option

To dockerize Liquibase with MySQL, you will need to follow these steps:

Install Docker on your machine if you don't have it already.
Create a Dockerfile for your Liquibase project. This file will contain the instructions for building a Docker image for your project.
Here is an example Dockerfile that you can use as a starting point:

Copy code
FROM openjdk:8-jdk-alpine

RUN apk update && apk add mysql-client

# Install Liquibase
RUN wget https://github.com/liquibase/liquibase/releases/download/liquibase-parent-3.10.3/liquibase-3.10.3-bin.tar.gz
RUN tar -xvf liquibase-3.10.3-bin.tar.gz
RUN mv liquibase-3.10.3 /usr/local/liquibase

# Set environment variables
ENV LIQUIBASE_HOME /usr/local/liquibase
ENV PATH $PATH:$LIQUIBASE_HOME

# Add the Liquibase scripts directory to the path
ENV PATH $PATH:$LIQUIBASE_HOME/bin

# Set the default command to run when starting the container
CMD ["liquibase"]
Build the Docker image using the Dockerfile. Run the following command in the same directory as the Dockerfile:
Copy code
docker build -t liquibase .
This will build a Docker image called "liquibase" using the instructions in the Dockerfile.

Start a MySQL container. You can do this using the following command:
Copy code
docker run --name mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:8
This will start a MySQL container with the name "mysql" and set the root password to "my-secret-pw".

Run the Liquibase container and link it to the MySQL container. Use the following command:
Copy code
docker run --name liquibase --link mysql:mysql -v /path/to/changelog:/changelog -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=my-secret-pw -d liquibase update
This will start a Liquibase container with the name "liquibase" and link it to the MySQL container. It will also mount the directory containing your changelog files to the "/changelog" directory in the container, and set the necessary environment variables for connecting to the MySQL database.

Finally, it will run the "update" command, which will apply the changes in the changelog to the database.

You can then use the Liquibase container to run other Liquibase commands as needed. For example, you can use the following command to rollback the changes:

Copy code
docker run --name liquibase --link mysql:mysql -v /path/to/changelog:/changelog -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=my-secret-pw -d liquibase rollbackCount 1
``
