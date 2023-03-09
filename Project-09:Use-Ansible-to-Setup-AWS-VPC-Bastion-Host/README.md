# Project-09: Use Ansible to Setup AWS VPC with Bastion Host

### Objective:

- Configuration Management of VPC: 
  - Using Ansible, setup the AWS VPC and its component (Subnets, Internet Gateway, NAT Gateway, Route Tables, NACL, Security Groups etc.) with Bastion host.

- Automatic Setup 

- Centralize Change Management

- Version Control 

### Architecture:

![GitHub Light](./snaps/pro09-ansible-vpc1.jpg)


### Flow of Execution:
- Login to AWS
- Create EC2 instance to create playbook
- Install ansible 
- Install boto 
- Setup EC2 role for ansible 
- Create project directory
- Sample cloud task (with key pair)
- Create variable file for VPC and bastion host
- Create VPC setup playbook
- Create Bastion host setup playbook
- Site.yml to call both playbook at once


### Step 1: Create EC2 insatnce for Ansible and IAM Role: 

- Login to AWS account
- Go to EC2 service 
- Click on Launch instance
  - Instane Name: ansible-control-plane
  - AMI: Ubuntu 20.04
  - Key Pair: ansible-control-plane
  - VPC: default
  - Security Group: Port 22 Access to MyIP
  - Userdata: 
  ```
  #!/bin/bash
  apt update
  apt install ansible -y 
  ```

- 