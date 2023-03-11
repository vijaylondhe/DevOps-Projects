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


### Step 2: Setup New Branch and Create VPC: 

- Pull the code from the Github
- git clone https://github.com/vijaylondhe/ansible-aws-vpc.git
- Create new branch `vprofile-stack`
  - git checkout -b vprofile-stack
  - git branch --all
- Move the content of `vars/bastion_setup` file to `vpc_setup` file
- Delete the `bastion_setup` file 
- Create new file `site.yml`, import the vpc and bastion playbook in it.

```
---
- import_playbook: vpc-setup.yml
- import_playbook: bastion-instance.yml
```
- Commit and Push the code to the GitHub

- Run the playbook
- `ansible-playbook site.yml`
- This will create VPC with all its component and bastion host.
- This will also create the `bastion-key.pem` file for bastion host access and `vars/output_vars` file for the output.
- Make sure to ignore key file while pushing our code to github.
- Create git ignore file, add .pem files and push to the github.
- vi .gitignore
```
*.pem
```
- git add .
- git commit -m "ignore keys"
- git push origin vprofile-stack


### Step 3: Create Playbook for EC2 stack:

#### 3.1 Setup variable files and key pair for the instance

- Locate the AMI ID which is required for EC2 instances.
- Go to the `https://cloud-images.ubuntu.com/locator/` website.
- Search for Ubuntu 18.04 image for AWS cloud `us-east-1` region.
- Copy the Image ID in below variable file.
- We will be using same AMI for all the EC2 instances.
- Create new variable file 
- `vi vars/vprostacksetup`
```
nginx_ami: ami-0263e4deb427da90e
tomcat_ami: ami-0263e4deb427da90e
memcache_ami: ami-0263e4deb427da90e
rmq_ami: ami-0263e4deb427da90e
mysql_ami: ami-0263e4deb427da90e
```

- Create new file for playbook
- `vi vpro-ec2-stack.yml`
```
- name: Setup Vprofile Stack
  hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: Import VPC setup variables
      include_vars: vars/vpc_setup

    - name: Import Vprofile setup variable
      include_vars: vars/vprostacksetup

    - name: Create Key pair
      ec2_key:
        name: vprokey
        region: "{{region}}"
      register: vprokey_out

    - name: Save private key into file loginkey_vpro.pem
      copy:
        content: "{{vprokey_out.key.private_key}}"
        dest: "./loginkey_vpro.pem"
        mode: 0600
      when: vprokey_out.changed
```
 
- Push the code to the github 
- git add .
- git commit -m "added playbook and var files for vprofile stack"
- git push origin vprofile-stack


- Run the playbook
- `ansible-playbook vpro-ec2-stack.yml`

![GitHub Light](./snaps/vprostack_key_pair_playbook.png)


#### 3.2 Create Security Groups for Load Balancer and EC2 Instances:

- Add `vars/output_vars` file inside the playbook
- Add Security Group for Load Balancer
- Note: If we execute the playbook multiple times, ec2_group module will update the security group rules everytime, by default it is not idempotent, to avoid this add `purge_rule: no` option in ec2_group module.


- vi vpro-ec2-stack.yml
```
    - name: Create Security Group for Load Balancer
      ec2_group:
        name: vproELB-sg
        description: Allow port 80 from everywhere and all port within sg
        region: "{{region}}"
        vpc_id: "{{vpcid}}"
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
      register: vproELBSG_out

    - name: Create Security Group for Vprofile stack
      ec2_group:
        name: vproStack-sg
        description: Allow port 80 from ELB SG and all port within sg
        region: "{{region}}"
        vpc_id: "{{vpcid}}"
        purge_rules: no
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            group_id: "{{vproELBSG_out.group_id}}"

          - proto: tcp
            from_port: 22
            to_port: 22
            group_id: "{{BastionSGid}}"
      register: vproStackSG_out

    - name: Update Security Group with its own sg id
      ec_group:
        name: vproStack-sg
        description: allow all ports within the sg
        region: "{{region}}"
        vpc_id: "{{vpcid}}"
        purge_rules: no
        rules:
          - proto: all
            group_id: "{{vproStackSG_out.group_id}}"

```