# Project-16: Managing AWS Infrastructure State with Terraform

### Objective:
Deploy and manage the state of the AWS Infrastructre using terraform.

### Prerequisites:
- Install Terraform and AWS CLI on the local machine 

### Architecture:

![GitHub Light](./snaps/pro-16-terraform-aws.drawio1.png)

### Flow of Execution:
- Setup Terarform with Backend
- Setup VPC and it's component
- Provision Beanstalk Environment
- Provision Backend Services 
  - RDS
  - ElastiCache
  - Amazon MQ
- Setup Security Group, Key Pair, Bastion Host etc.


### Step 1: Create IAM User and S3 Bucket:
- Create IAM User: 
  - Go to the AWS Console -> IAM Service 
  - Create New User -> terradmin
  - Permissions -> AdministratorAccess
  - Generate the Access Keys

- Create S3 Bucket to Store the State file:
  - Go to S3 Service
  - Create Bucket -> terraform-state-bucket-815
  - Region -> us-east-1
  - Block all public access

### Step 2: Create Git Repository for Terraform Code:
- Go to the GitHub
- Create New Repository
- Repository Name -> terraform-aws-vprofile
- Login to local machine 

```
mkdir terraform-aws-vprofile
cd terraform-aws-vprofile
git init 
echo "terraform project" >> README.md 
git add . 
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/vijaylondhe/terraform-aws-vprofile.git
git push -u origin main

```





