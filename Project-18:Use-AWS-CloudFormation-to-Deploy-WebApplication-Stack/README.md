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
