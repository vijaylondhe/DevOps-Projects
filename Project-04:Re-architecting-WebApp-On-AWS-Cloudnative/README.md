# Project-04: Re-architecting Web App On AWS (Cloudnative)

In this project, we will use AWS managed services to make the web application as cloud native application. 

Following AWS services will be used:

- Elastic Beanstalk

- RDS

- Elasticache

- Active MQ

- Route53

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
  - Create

- Create Parameter Group:
  - 
