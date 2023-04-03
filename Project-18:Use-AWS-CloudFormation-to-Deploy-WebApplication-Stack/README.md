# Project-18: Use AWS CloudFormation to Deploy Web Application Stack

### Objective:
Using AWS Cloudformation, deploy the web application stack on the AWS.

### Tools & Services:
- AWS CloudFormation
- AWS S3 

<!-- ### Architecture: -->

<!-- ![GitHub Light](./snaps/.png) -->

### Flow of Execution:
- Create S3 bucket to upload the templates, create folder named stack-template 
- Create Key Pair 
- Write root template named cicdtemp.yml
- Write Child templates 
  - Create S3 Role
  - Jenkins Template
  - Nexus Template 
  - Sonar Template
  - Windows Server Template
  - Tomcat Template
  - DB Template 
- Update root template with child template paths 
- Upload all child template to the S3 bucket in the folder stack-template
- Create Nested Template by using cicdtemp.yml

### Step 1: Create S3 Bucket and Key Pair:
- Create Bucket:
  - Go to the S3 Service 
  - Click on Create bucket

```
Region Name: us-east-1 
Bucket Name: vprofile-cicd-template-815
Create Folder inside the bucket
Folder Name: stack-template
```

![GitHub Light](./snaps/pro-18-s3-bucket.png)

- Create Key Pair:
  - Go to the EC2 Service -> Key Pair
  - Create New Key
  - Key Name: cicd-stack-key
  - Format: .pem 

![GitHub Light](./snaps/pro-18-key-pair.png)

### Step 2: Create Root Template:

- vi cicdtemp.yml
```
AWSTemplateFormatVersion: "2010-09-09"
Description: Root Template
Parameters:
  KeyPair:
    Description: Key Pair for the Instance
    Type: AWS::EC2::KeyPair::KeyName
  MyIP:
    Description: Assigning IP
    Type: String
    Default: 0.0.0.0/0

Resources:
  S3RoleforCicd:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/cicds3role.yaml

  JenkinsInst:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/jenk.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIp: !Ref MyIP

  App01qa:
    Type: AWS::CloudFormation::Stack
    DependsOn: JenkinsInst
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/app01qa.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIP: !Ref MyIP
  NexusServer:
    Type: AWS::CloudFormation::Stack
    DependsOn: JenkinsInst
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/nexus.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIP: !Ref MyIP
  SonarServer:
    Type: AWS::CloudFormation::Stack
    DependsOn: JenkinsInst
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/sonar.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIP: !Ref MyIP
  db01qa:
    Type: AWS::CloudFormation::Stack
    DependsOn: App01qa
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/db01qa.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIP: !Ref MyIP
  WintestServer:
    Type: AWS::CloudFormation::Stack
    DependsOn: JenkinsInst
    Properties:
      TemplateURL: https://s3.amazonaws.com/vpro-cicd-templates-815/stack-template/wintest.yaml
      Parameters:
        KeyName: !Ref KeyPair
        MyIP: !Ref MyIP
```

### Step 3: Create CF Template for IAM Role:
- Create Cloudformation template for IAM role 
- This IAM Role has full access to S3 service 
- Also, this role includes the instance profile 
- We will use the Cloudformation `Outputs` to export the IAM role which will later used to assing to the other templates 

- vi cicds3role.yaml
```
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  VPS3Role:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          -
            Effect: "Allow"
            Principal:
              Service:
                - "ec2.amazonaws.com"
            Action:
              - "sts:AssumeRole"
      Path: "/"
      Policies:
        Type: "AWS::IAM::Policy"
        Properties:
          PolicyName: "root"
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: "Allow"
                Action: "*"
                Resource: "arn:aws:s3:::*"
  VPInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Path: "/"
      Roles:
        - !Ref VPS3Role

Outputs:
  VPs3rolDetails:
    Description: VP CICD Pro s3 role info
    Value: !Ref VPInstanceProfile
    Export:
      Name: cicds3role-VPS3RoleProfileName
```

