# Project-06: Continuous Integration with AWS

### Objectives:

- In this project, instead of jenkins as countinous integration server we will use AWS managed services to build and test our source code. 

- This will reduce the operational overhead to manage the servers deployed for jenkins, nexus repository and sonarqube.


### Tools/Services Used:

- AWS CodeCommit
- AWS CodeBuild
- AWS CodeArtifact
- AWS CodePipeline
- AWS Systems Manager
- AWS S3 
- SonarCloud

### Architecture:

![GitHub Light](./snaps/pro-06-ci-with-aws.jpg)


### Flow of Execution:

- Login to AWS account
- CodeCommit:
  - Create CodeCommit Repo
  - Create IAM user with CodeCommit policy
  - Generate SSH Keys locally
  - Exchange keys with IAM user
  - Put source code from github repo to CodeCommit repo and push
- CodeArtifact:
  - Create IAM user with CodeArtifact access
  - Install AWS CLI and configure
  - Export auth token
  - Update settings.xml file in source code top level directory with below details
  - Update pom.xml file with repo details 
- Sonar Cloud:
  - Create sonar cloud account 
  - Generate the token 
  - Create SSM paramter with sonar details 
  - Create Build project 
  - Update CodeBuild role to access SSM parameter store
- Create notifications for sns or slack
- Build Project:
  - Update pom.xml with artifact version with timestamp
  - Create variables in SSM => parameter store
  - Create build project 
  - Update codebuild role to access SSM parameter store
- Create Pipeline:
  - CodeCommit
  - Testcode
  - Build
  - Deploy to S3 bucket
- Test the Pipeline
