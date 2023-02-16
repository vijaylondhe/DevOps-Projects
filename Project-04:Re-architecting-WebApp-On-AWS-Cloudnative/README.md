# Project-04: Re-architecting Web App On AWS (Cloudnative)

In this project, we will use AWS managed services to make the web application as cloud native application. 

Following AWS services will be used:

- Elastic Beanstalk

- RDS

- Elasticache

- Active MQ

- Route53

- S3

- CloudFront 


## Flow of Execution

- Login to AWS account 
- Create key pair for Beanstalk, RDS instance and Active MQ
- Create RDS Instance
- Configure AWS Elasticache
- Configure AWS Active MQ
- Create Elastic Beanstalk environment
- Update SG of backend to allow traffic from beanstalk SG
- Update SG of backend to allow internal traffic
- Launch the EC2 instance for DB initialization
- Login to instance and initialize RDS database
- Change healthcheck on beanstalk to /login
- Add 443 https listner on ELB
- Build artifact with the Backend Information
- Deploy the artifact to beanstalk 
- Create CDN with SSL cert
- Update the entry in GoDaddy DNS zones 
- Test the URL


## Architecture 

![GitHub Light](./snaps/cloud_native_app_aws.jpg)


## Steps:

### Create Key Pair 

- Create key pair to login to Elastic Beanstalk Instance. 
- Key Name: vprofile-bean-key
- Key Format: .pem 
- Region: N. Virginia


### Create Security Group for Backend Services

- Security Group Name: vprofile-backend-sg
- Description: Security Group for backend services 
- Inbound Rules: Custom TCP 22 Port Allow from MyIP 
- Save the Rules.
- Edit the Security Group again
- Add Inbound Rule: Allow all traffic for the same security group id
- Note: This will allow all backend services to communicate with each other
- Also delete rule added for the port 22
- Save the Rules

### Create RDS MySQL Instance

- Go to RDS service

- Create DB subnet group:
  - Name: vprofile-rds-subnet-group
  - Description: DB subnet group for vprofile MySQL RDS instance
  - VPC: Default
  - Add Subnets: Select all the availability zones and all the subnets in the az.
  - Click on Create

- Create Parameter Group:
  - Parameter Group Details
  - Parameter Group Family: mysql5.7
  - Type: DB Parameter Group
  - Group Name: vprofile-rds-parameter-group
  - Description: parameter group for MySQL RDS instance
  - Click on Create
  - Note: This parameter group created with default parameters, you can edit the required parameters. 

- Now, we have security group, DB subnet group and parameter group, so lets create RDS instance.

- Create RDS Instance: 
  - Click on Databases, create database
  - Database creation method: Standard Create 
  - Engine Type: MySQL 
  - Version: MySQL 5.7.22
  - Templates: Dev/Test
  - DB Instance Identifier: vprofile-rds-mysql
  - Master username: Admin
  - Password: Auto generated 
  - DB Instance Class: Burstable Classes (db.t3.micro)
  - Storage: gp2 20GB
  - Availability & Durability: Do not create stand by instance 
  - VPC: default
  - Subnet Group: vprofile-rds-subnet-group
  - Public Access: No
  - Security Group: vprofile-backend-sg
  - Database Port: 3306 
  - Database Authentication: Password Authentication
  - Additional Configuration
  - Initial Database Name: accounts
  - DB Parameter Group: vprofile-rds-parameter-group
  - Enable Automatic Backup: Yes (Backup Retention: 7 days)
  - Backup Window: No Preference
  - Monitoring: Disable Enhanced Monitoring
  - Enable delete protection 
  - Click on Create Database
  - Click on View Credential Detail (Copy this credential in text file, to use it later in application.properties file)


### Configure ElastiCache 

- Create Parameter Group:
  - Name: vprofile-memcached-parameter-group
  - Description: vprofile-memcached-parameter-group
  - Family: Memcached1.4
  - Click on Create 

- Create Subnet Group:
  - Click on Create Subnet Group 
  - Name: vprofile-memcached-subnet-group
  - Description: vprofile-memcached-subnet-group
  - VPC: default
  - Select all the subnet in all availability zones 
  - Click on Create 

- Create Cluster:
  - Click on Get Started 
  - Create Cluster -> Create Memcached Cluster
  - Location: AWS Cloud
  - Name: vprofile-elasticache-service 
  - Description: vprofile-elasticache-service 
  - Engine Version: 1.4.5
  - Port: 11211
  - Parameter Groups: vprofile-memcached-parameter-group
  - Node Type: cache.t2.micro
  - Number of Nodes: 1 
  - Subnet Group Setting: vprofile-memcached-subnet-group
  - Availability Zone: No Prefernce
  - Security Group: vprofile-backend-sg
  - Tags: Name: vprofile-elasticache-service 
  - Click on Create 


### Configrue Amazon MQ

- Go to Amazon MQ service
- Click on Get Started 
- Broker Engine Type: RabbitMQ
- Deployment Mode: Single-instance Broker 
- Broker Name: vprofile-rmq
- Broker Instance Type: mq.t3.micro
- RabbitMQ Access:
- Username: rabbit
- Password: *****
- Broker Engine Version: 3.9.16
- Network and Security: Access Type -> Private Access
- VPC and Subnet: Use default VPC and subnets
- Security Group: vprofile-backend-sg
- Click on Create

### Initialize the Database

- Lauch new EC2 instance
- Instance Name: mysql-client
- AMI: Ubuntu 18.04
- Instance Type: t2.micro
- Key Pair: vprofile-bean-key
- VPC: default
- Security Group: Create new SG -> Name: mysqlclient-sg Port-> 22 From -> MyIP
- User Data:

```
#!/bin/bash
sudo apt update
sudo apt install mysql-client -y
```

- Launch the instance
- Edit the SG of Backend Service and Add the access for mysql-client instance
- Port 3306 -> SG (mysqlclient-sg)
- Get the instance public ip and login into it 
- Clone the repository

```
git clone https://github.com/vijaylondhe/vprofile-project.git
cd vprofile-project/src/main/resources
mysql -h <rds_endpoint> -u admin -p<rds_password> accounts < db_backup.sql
```

- Login to mysql database and check db is initialzed or not 

```
mysql -h <rds_endpoint> -u admin -p<rds_password>
```

```
show databases;
show tables;
```


### Create Elastic Beanstalk

- Go to Elastic beanstalk service
- Click on Create Application
- Application Name: vprofile-java-app
- Tags: Project -> vprofile
- Platform: Tomcat 
- Platform Branch: Tomcat 8 with Correto 11 running on 64 bit Amazon Linux 2
- Platform Version: 4.2.16
- Application Code: Sample Application 
- Click on Configure more options 
- Presets: Custom Configuration
- Instances: 
  - Security group: vprofile-backend-sg
  - Root Volume: Container default 
- Capcity: 
  - Auto Scaling Group: Load Balanced 
  - Instances: Min. 2, Max. 4 
  - Instance Types: t2.micro
  - Availability Zone: Any 
  - Scaling Triggers: Metric -> Network Out (Keep default setting)
- Load Balancer (will do this later)
- Rolling Updates & Deployments
  - Deployment Policy: Rolling
  - Batch Size: 50%
- Security:
  - Key Pair: vprofile-bean-key
  - IAM Instance Profile: Create Instance Profile 
- Monitoring:
  - System: Enhanced
- Tags: Project -> vprofile
- Click on Create App


### Update the Security Group and ELB

- In EC2 service check the instances created by Elastic beanstalk 
- Check the security group id attached to this instances 
- Edit the backend services security group i.e vprofile-backend-sg
- Add the below rules
  - Custom TCP -> Port 3306  -> Source (SG id of elastic beanstalk instances)
  - Custom TCP -> Port 11211 -> Source (SG id of elastic beanstalk instances)
  - Custom TCP -> Port 5671  -> Source (SG id of elsatic beanstalk instances)
- Save the rule 

- Go to Elastic beanstalk service, click on environments 
- Select the Configuration -> Load Balancer (Edit)
- Add one more listenr for Port 443 HTTPS ->  Choose the certificate *.cloudndevops.in
- Edit the Processes -> Change health check path to /login 
- In Sessions, enable the stickyness 
- Click on Apply


### Build & Deploy Artifact

- Login to the local machine 
- Clone the source code 

```
git clone https://github.com/vijaylondhe/vprofile-project.git
cd vprofile-project
```

- Edit the application.properties file and update the backend service endpoints 

```
cd src/main/resources
vim application.properties

#JDBC Configutation for Database Connection
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://<rds_endpoint>:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=admin
jdbc.password=<rds_password>

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=<Elasticache_endpoint_remove_port_number>
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=<rabbit_mq_endpoint_remove_port_number>
rabbitmq.port=5671
rabbitmq.username=rabbit
rabbitmq.password=<password>

#Elasticesearch Configuration
elasticsearch.host =192.168.1.85
elasticsearch.port =9300
elasticsearch.cluster=vprofile
elasticsearch.node=vprofilenode
```

- Build the artifact 

```
cd ../../..
mvn install
```

- Check artifact is created inside the /target directory

```
ls /target
```

- In Elastic Beanstalk service, go to the environments 
- Click on Application Versions 
- Click on Upload 
- Version label: vprofile-v2.5 
- Choose file from the /target directory
- Click on Upload
- Once it is uploaded, select the version: vprofile-v2.5 
- Click on Actions -> Deploy -> Select the environment (vprofile-env)
- In Environment, go to the Events 
- Check the event you will see, environment update is starting 
- As we configured rolling update deployment type, so you will the updates will happens in batch-1 and batch-2

- Update the DNS in GoDaddy
- Add new record
- Type -> CNAME, Host -> vprofile, Points-to -> EB_endpoint
- In Elastic beanstalk, in Load Balancer make sure stickyness policy is enabled 

- In Browser, test the application.

```
https://vprofileapp.cloudndevops.in/login
```


### Setup CloudFront

- Go to cloufront service
- Click on Create a Cloudfront distribution
- Origin Domain: vprofileapp.cloudndevops.in
- Protocol: Match viewer
- Origin Path: keep as it is 
- Viewer Protocol Policy: HTTP and HTTPS
- Allowed HTTP Methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE 
- Alternate domain name: vprofileapp.cloudndevops.in
- Custom SSL certificate: select certificated from ACM
- TLS Policy: TLSv1
- Create distribution


### Verify the Application

- In browser type the application url

```
https://vprofileapp.cloudndevops.in
```