# Project-03: Lift and Shift the WebApplication on AWS 

In this project we will setup the web application stack on the AWS cloud.

Following AWS services will be used 

- EC2
- ELB
- Auto Scaling 
- EBS/EFS
- Route53
- IAM
- ACM
- S3 

## Flow of execution:

- Login to the AWS account.
- Create key pair to access the instances.
- Create Security groups, so that only required ports will be opened on the instances. 
- Launch the EC2 instances with userdata.
- Update the IP addresses in Route53. 
- Build the application from the source code.
- Upload the artifact to S3 storage.
- Download the artifact from S3 to EC2 tomcat instance.
- Setup the ELB with HTTPS (Use ACM for SSL certifiacte).
- Map the ELB endpoint url to website name in DNS provider (e.g. GoDaddy).
- Verify the application. 


## Architecture 

![GitHub Light](./snaps/lift_and_shift_to_aws.jpg)


### Create Certificate in AWS Certificate Manager (ACM)

- Login to the AWS account and select N.Virginia as region

![GitHub Light](./snaps/Login.png)

- Go to Certificate Manager service and click on Request a Certificate 

![GitHub Light](./snaps/ACM_service.png)

- Select certificate type as public certificate and click next

![GitHub Light](./snaps/Request_cert.png)

- Enter the domain name and DNS validation method as show in below and click on Request 

![GitHub Light](./snaps/domain_name_validation.png)

- You will see Certificate ID and the status as "pending"

- Click on Certificate ID copy the CNAME name and CNAME value and create CNAME record in your DNS provider (e.g. GoDaddy)

- After some time you will see the certifiacte status as "Issued"

![GitHub Light](./snaps/Issued_status.png)


### Create Key Pair 

Select N.virgina as region

Go to EC2 service and under the Network and Security select the Key Pairs 

Provide name to keypair e.g. vprofile-key select .pem as private key file format


### Create Security Groups

- Create Security group for Application load balancer 

In inbound rules add 80 and 443 port for all internet access, so that our application will be accessed from the internet.

![GitHub Light](./snaps/SG_ALB.png)

- Create Security group for Apache tomcat instance 

In inbound rules add port 8080 and allow access from security group of ALB.

![GitHub Light](./snaps/SG_Apache_tomcat.png)

- Create Security group for backend services (RabbitMQ, Memcached, MySQL)

In inbound rules add port 3306 (MySQL), 11211 (Memcached), 5672 (RabbitMQ) and allow access to security group of Apache Tomcat only.

Also add one inbound rule with ALL traffic with self Security Group ID (This will allow all backend services to communicate with each other). 

![GitHub Light](./snaps/SG_Backend_Services.png)


