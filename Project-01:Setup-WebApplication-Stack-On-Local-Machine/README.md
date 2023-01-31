# Project-01: Setup Web Application Stack on Local Machine 

In this project we will setup the Web Application Stack on the local machine using, this stack includes following tools and services.

- NGINX: This will be act as load balancer and forward user requests to the Apache Tomcat application server.

- Apache Tomcat: This will be our application server to deploy the java web application.

- RabbitMQ: Used as message broker for the application.

- Memcached: Used to cache data from the MySQL database server for faster user performance. 

- MySQL: Database server for the application.


## Pre-requisite 

- Oracle Virtual Box (for Windows)/ VMware Fusion Tech (for MacOS)
- Vagrant (for VM automation)
- Vagrant Plugin (vagrant-hostmanager/vagrant-vmware-desktop) 
- Git
- JDK8
- Maven 
- IDE (VSCode)


## Architecture 

![GitHub Light](/snaps/web-app-stack-local.png)

## Step 1: Setup Virtual Machines

Clone the repository

```
git clone https://github.com/vijaylondhe/vprofile-project.git
```