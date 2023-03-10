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
  - Key Pair: Create New, Name: ansible-control-plane
  - VPC: default
  - Security Group: Port 22 Access to MyIP
  - Userdata: 
  ```
  #!/bin/bash
  apt update
  apt install ansible -y 
  apt install awscli -y 
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


### Step 2: Create Sample Playbooks:

- Login to EC2 instance
- Install boto packages 
```
apt install python3-boto -y
apt install python3-boto3 -y
apt install python3-botocore -y
```

- Create separate directory for playbooks 

```
mkdir vpc-stack-vprofile
cd vpc-stack-vprofile
```

- Create sample playbook
- `vim test-aws.yml`

```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: sample keypair 
      ec2_key:
       name: my-keypair
       region: us-east-1 
```

- Run the playbook, this will create the key pair 
- `ansible-playbook test-aws.yml`

- Go to the AWS console in EC2 service 
- Click on Key Pair and delete the sample-key 
- We will store the key in file using copy module 
- `vim test-aws.yml`
```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: create sample key 
      ec2_key: 
        name: sample-key
        region: us-east-1
      register: keyout

    - debug:
        var: keyout 

    - name: store login key 
      copy: 
        content: "{{keyout.key.private_key}}"
        dest: ./sample-key.pem
      when: keyout.changed

```

- Run the playbook
- `ansible-playbook test-aws.yml`


### Step 3: Create Variable files 

- Create GIt repository to store the ansible playbooks and use any IDE to edit the ansible playbooks 

- Repository reference: `https://github.com/vijaylondhe/ansible-aws-vpc.git`

- Create variable file for vpc setup playbook
- vim vars/vpc_setup
```
vpc_name: "Vprofile-vpc"

#VPC Range
vpcCidr: '10.0.0.0/16'

#Subnet Range
PubSub1Cidr: 10.0.1.0/24
PubSub2Cidr: 10.0.2.0/24
PubSub3Cidr: 10.0.3.0/24
PrivSub1Cidr: 10.0.4.0/24
PrivSub2Cidr: 10.0.5.0/24
PrivSub3Cidr: 10.0.6.0/24

#Region Name
region: "us-east-1"

#Zone Names
zone1: us-east-1a
zone2: us-east-1b
zone3: us-east-1c

state: present
```

- Create variable file for bastion host playbook
- vim vars/bastion_setup
```
bastion_ami: ami-006dcf34c09e50022
region: us-east-1
MYIP: 0.0.0.0/32
keyName: vprofile-key
instanceType: t2.micro
```

### Step 4: Create Playbook for VPC:

- Create playbook vpc-setup.yml
- vim vpc-setup.yml
```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: Import vpc variables
      include_vars: vars/vpc_setup

    - name: Create vprofile VPC
      ec2_vpc_net:
        name: "{{ vpc_name }}"
        cidr_block: "{{ vpcCidr }}"
        region: "{{ region }}"
        dns_support: yes
        dns_hostnames: yes
        tenancy: default
        state: "{{ state }}"
      register: vpcout

    - debug:
        var: vpcout
```

- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook vpc-setup.yml`

### Step 5: Update Playbook for Private and Public Subnets:
- vim vpc-setup.yml
```
    - name: Create Public Subnet 1 in Zone1
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone1 }}"
        state: "{{ state }}"
        cidr: "{{ PubSub1Cidr }}"
        map_public: yes
        resource_tags:
          Name: vprofile-pubsub1
      register: pubsub1_out

    - name: Create Public Subnet 2 in Zone2
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone2 }}"
        state: "{{ state }}"
        cidr: "{{ PubSub2Cidr }}"
        map_public: yes
        resource_tags:
          Name: vprofile-pubsub2
      register: pubsub2_out

    - name: Create Public Subnet 3 in Zone3
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone3 }}"
        state: "{{ state }}"
        cidr: "{{ PubSub2Cidr }}"
        map_public: yes
        resource_tags:
          Name: vprofile-pubsub3
      register: pubsub3_out

    - name: Create Private Subnet 1 in Zone1
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone1 }}"
        state: "{{ state }}"
        cidr: "{{ PrivSub1Cidr }}"
        resource_tags:
          Name: vprofile-privsub1
      register: privsub1_out

    - name: Create Private Subnet 2 in Zone2
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone2 }}"
        state: "{{ state }}"
        cidr: "{{ PrivSub2Cidr }}"
        resource_tags:
          Name: vprofile-privsub2
      register: privsub2_out

    - name: Create Private Subnet 3 in Zone3
      ec2_vpc_subnet:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        az: "{{ zone3 }}"
        state: "{{ state }}"
        cidr: "{{ PrivSub3Cidr }}"
        resource_tags:
          Name: vprofile-privsub3
      register: privsub3_out
```

- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook vpc-setup.yml`

### Step 6: Update Playbook for Internet Gateway and Public Route Table:

- vim vpc-setup.yml
```
    - name: Internet Gateway Setup
      ec2_vpc_igw:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        state: "{{ state }}"
        resource_tags:
          Name: vprofile_IGW
      register: igw_out


    - name: Setup Public subnet Route Table
      ec2_vpc_route_table:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        tags:
          Name: Vprofile-PubRT
        subnets:
          - "{{pubsub1_out.subnet.id}}"
          - "{{pubsub2_out.subnet.id}}"
          - "{{pubsub3_out.subnet.id}}"
        routes:
          - dest: 0.0.0.0/0
            gateway_id: "{{igw_out.gateway_id}}"
      register: pubRT_out

```
- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook vpc-setup.yml`

### Step 7: Update Playbook for NAT Gateway and Private Route Table:

- vim vpc-setup.yml
```
    - name: Create new NAT Gateway1 and allocate new EIP if NATGW does not exist yet in the subnet
      ec2_vpc_nat_gateway:
        state: "{{ state }}"
        subnet_id: "{{ pubsub1_out.subnet.id }}"
        wait: yes
        region: "{{ region }}"
        if_exist_do_not_create: true
      register: NATGW_out


    - name: Setup Private subnet route table
      ec2_vpc_route_table:
        vpc_id: "{{ vpcout.vpc.id }}"
        region: "{{ region }}"
        tags:
          Name: Vprofile-PrivRT
        subnets:
          - "{{privsub1_out.subnet.id}}"
          - "{{privsub2_out.subnet.id}}"
          - "{{privsub3_out.subnet.id}}"
        routes:
          - dest: 0.0.0.0/0
            gateway_id: "{{NATGW_out.gateway_id}}"
      register: privRT_out

```

- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook vpc-setup.yml`

### Step 8: Update Playbook to set facts and create output file:

- vim vpc-setup.yml
```
    - debug:
        var: "{{item}}"
      loop:
        - vpcout.vpc.id
        - pubsub1_out.subnet.id
        - pubvsub2_out.subnet.id
        - pubvsub3_out.subnet.id
        - privsub1_out.subnet.id
        - privsub3_out.subnet.id
        - privsub3_out.subnet.id
        - igw_out.gateway_id
        - NATGW_out.gateway_id
        - pubRT_out.route_table.id
        - privRT_out.route_table.id

    - set_fact:
        vpcid: "{{vpcout.vpc.id}}"
        pubsub1id: "{{pubsub1_out.subnet.id}}"
        pubsub2id: "{{pubsub2_out.subnet.id}}"
        pubsub3id: "{{pubsub3_out.subnet.id}}"
        privsub1id: "{{privsub1_out.subnet.id}}"
        privsub2id: "{{privsub2_out.subnet.id}}"
        privsub3id: "{{privsub3_out.subnet.id}}"
        igwid: "{{igw_out.gateway_id}}"
        pubRTid: "{{pubRT_out.route_table.id}}"
        privRTid: "{{privRT_out.route_table.id}}"
        NATGWid: "{{NATGW_out.gateway_id}}"
        cacheable: yes

    - name: Create varibales file for vpc output
      copy:
        content: "vpcid: {{vpcout.vpc.id}}\npubsub1id: {{pubsub1_out.subnet.id}}\npubsub2id: {{pubsub2_out.subnet.id}}\npubsub3id: {{pubsub3_out.subnet.id}}\nprivsub1id: {{privsub1_out.subnet.id}}\nprivsub2id: {{privsub2_out.subnet.id}}\nprivsub3id: {{privsub3_out.subnet.id}}\nigwid: {{igw_out.gateway_id}}\npubRTid: {{pubRT_out.route_table.id}}\nprivRTid: {{privRT_out.route_table.id}}\nNATGWid: {{NATGW_out.gateway_id}}\n"
        dest: vars/output_vars
```

- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook vpc-setup.yml`

### Step 9: Create Playbook for Bastion Host Setup

- Create playbook `bastion-instance.yml`
- Make sure to include the variable files 
- We will use the output id from the variable file to input in this playbook
- vim bastion-instance.yml
```
- host: localhost
  connection: local
  gather_facts: False
  tasks:
    - name: Import VPC setup variable
      include_vars: vars/bastion_setup

    - name: Import VPC setup variable
      include_vars: vars/output_vars

    - name: Create vprofile ec2 key
      ec2_key:
        name: vprofile-key
        region: "{{region}}"
      register: key_out

    - name: Save private key into file bastion-key.pem
      copy:
        content: "{{key_out.key.private_key}}"
        dest: "./bastion-key.pem"
        mode: 0600
      when: key_out.changed

    - name: Create security group for bastion host
      ec2_group:
        name: Bastion-host-sg
        description: Allow port 22 from everywhere and all port within sg
        region: "{{region}}"
        vpc_id: "{{vpcid}}"
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: "{{MYIP}}"
      register: BastionSG_out


    - name: Create Bastion Host
      ec2:
        key_name: vprofile-key
        region: "{{region}}"
        instance_type: t2.micro
        image: "{{bastion_ami}}"
        wait: yes
        wait_timeout: 300
        instance_tags:
          Name: "Bastion_host"
          Project: Vprofile
          Owner: DevOps Team
        exact_count: 1
        count_tag:
          Name: "Bastion_host"
          Project: Vprofile
          Owner: DevOps Team
        group_id: "{{BastionSG_out.group_id}}"
        vpc_subnet_id: "{{pubsub1id}}"
      register: bastionHost_out
```
- Commit and push the changes to the github
- Run the playbook
- `ansible-playbook bastion-instance.yml`

- Go to the AWS console and check new instance is provisioned for Bastion Host.

- Access the instance from the key file `bastion-key.pem`