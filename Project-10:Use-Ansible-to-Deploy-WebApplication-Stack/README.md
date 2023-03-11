# Project-10: Use Ansible to Deploy Web Application Stack on AWS

### Objective:

- Setup VPC [Secure and HA] (Refer Previous Project-09)
- Provision EC2 instances, ELB, Security Groups etc.
- Provision Web Application Stack on EC2 instances
  - Build Artifact
  - MySQL
  - Memcached
  - RabbitMQ
  - Tomcat
  - Nginx 

### Flow of Execution:
- Login to the AWS
- Create EC2 instance to run ansible playbook
- Install Ansible
- Install Boto
- Setup IAM Role for ansible and attach to EC2
- Fetch source code from project ansible for AWS
- Execute VPC playbook
- Playbook to launch EC2, ELB, Security groups for web application stack
- Get into VPC
- Playbooks for web application stack


### Step 1: Create EC2 insatnce for Ansible and IAM Role: 

- Login to AWS account
- Go to EC2 service 
- Click on Launch instance
  - Instane Name: ansible-control-plane
  - AMI: Ubuntu 20.04
  - Key Pair: Create New, Name: ansible-control-plane
  - VPC: default
  - Security Group: Port 22 Access to MyIP
  - Userdata: 
  ```
  #!/bin/bash
  apt update
  apt install ansible -y 
  apt install awscli -y 
  apt install python3-boto -y
  apt install python3-boto3 -y
  apt install python3-botocore -y
  ```

- Create IAM Role:
  - Go to IAM Service
  - Create Role 
  - Trusted entity type: AWS Service 
  - Use Case: EC2
  - Add Permissions: AdministratorAccess
  - Role Name: ansible-admin-role
  - Click on Create Role

- Attach Role to EC2 instance.
  - Go to the EC2 service 
  - Select the EC2 instance
  - Go to Actions -> Security -> Modify IAM Role 
  - Choose ansible-admin-role
  - Click on Update IAM Role 

- Log in to the EC2 instance and verify the installation 
  - Check IAM role is attached or not  
  - Run the command `aws sts get-caller-identity` 
  - Check ansible is installed or not 
  - Run the command `ansible --version`


### Step 2: Setup New Branch and Site.yml file: 

- Pull the code from the Github
- git clone https://github.com/vijaylondhe/ansible-aws-vpc.git
- Create new branch `vprofile-stack`
  - git checkout -b vprofile-stack
  - git branch --all
- Move the content of `vars/bastion_setup` file to `vpc_setup` file
- Delete the `bastion_setup` file 
- Create new file `site.yml`
```
---
- import_playbook: vpc-setup.yml
- import_playbook: bastion-instance.yml
```
- Commit and Push the code to the GitHub



