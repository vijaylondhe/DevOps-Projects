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
- AWS SNS
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


### Step 1: Create CodeCommit Repository and IAM User:

- Login to AWS Console 
- Go to CodeCommit Service
  - Create New Repository
  - Name: `vprofile-code-repo`

![GitHub Light](./snaps/codecommit_repo.png)

- Create IAM role for CodeCommit:
  - Click on create new role
  - Add Permissions => Create Policy
  - Choose Service => CodeCommit 
  - In Actions => Click on All CodeCommit actions 
  - In Resources => Select Specific => Add ARN (Provide Region and Repository Name)

    ![GitHub Light](./snaps/iam_role_arn.png)

  - Click on Add
  - Click on Tags => Review 
  - Policy Name: `vprofile-code-admin-policy`
  - Click on Create Role
  - Attach new policy: `vprofile-code-admin-policy`
  - Click Next 
  - Provide Role Name: `vprofile-code-admin-role`
  - Click on Create Role

![GitHub Light](./snaps/iam_role.png)


- Create IAM user 
  - Create user
  - Name: `vprofile-code-admin`
  - Attach Policy `vprofile-code-admin-policy`

![GitHub Light](./snaps/iam_user.png)

- Create SSH keys in local system
  - Execute `ssh-keygen` command
  - Provide the path for the keys 

![GitHub Light](./snaps/create_ssh_keys.png)

- Upload SSH keys to IAM user
  - Go to IAM user
  - Select the user `vprofile-code-admin`
  - Go to security credentials
  - Upload the keys in SSH Public Keys for CodeCommit
  - Copy the content of `.ssh/vprofile-repo_id_rsa.pub` and paste it.

![GitHub Light](./snaps/upload_ssh_keys.png)

- Create config file for the codecommit
  - Go to `cd ~/.ssh`
  - Create file `vim config`
  ```
  Host git-codecommit.us-east-1.amazonaws.com
    User <SSH_Key_ID_from IAM_user>
    IdentityFile ~/.ssh/vprofile-repo_id_rsa
  ```

  - Test the SSH connection with CodeCommit
  ```
  ssh git-codecommit.us-east-1.amazonaws.com
  ```
![GitHub Light](./snaps/test_ssh_repo_connection.png)


- Clone the github repository in local and convert this repository to codecommit repository

- Use this repository location: `https://github.com/devopshydclub/vprofile-project.git` 

```
git checkout master
git branch -a | grep -v HEAD | cur -d'/' -f3 | grep -v master > /tmp/branches
for i in `cat  /tmp/branches`; do git checkout $i; done
git fetch --tags
git remote rm origin
git remote add origin ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/vprofile-code-repo
cat .git/config
git push origin --all
git push --tags
```
- Go to the CodeCommit service and check `vprofile-code-repo` repository have added files from our local repository.

![GitHub Light](./snaps/repo_after_push.png)


### Step 2: Setup CodeArtifact:

create repo
create user with code artifact poilcy
install aws cli on local machine
run aws configure
go to code artifact and check connection settings for maven 
update settings.xml
update pom.xml
commit the code to codecommit on ci-aws branch 


### Step 3: Setup SonarCloud Account:

create account on sonarcloud 
create token
create project manually 

### Step 4: Setup SSM Parameter Store:

create parameter store in systems manager
create organization name 
create host https://sonarcloud.io
project vprofile-repo
sonartoken securestring 
codeartifact-token securestring

### Step 5: Setup CodeBuild for Sonarqube Code Analysis:

modify code build role to access the parameter store
create iam policy and attach to the role 
create codebuild project 
add buidlspec file 
set cloudwatch logs for the build 

- Buildspec file for sonarqube code analysis

```
version: 0.2 
env: 
  parameter-store:
    LOGIN: sonartoken
    HOST: HOST
    Organization: Organization
    Project: Project
    CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN
phases:
  install:
    runtime-versions:
      java: corretto8
    commands:
    - cp ./settings.xml /root/.m2/settings.xml
  pre_build:
    commands: 
      - apt-get update 
      - apt-get install -y jq checkstyle
      - wget http://www-eu.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
      - tar xzf apache-maven-3.5.4-bin.tar.gz
      - ln -s apache-maven-3.5.4 maven
      - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
      - unzip ./sonar-scanner-cli-3.3.0.1492-linux.zip
      - export PATH=$PATH:/sonar-scanner-3.3.0.1492-linux/bin/
  build:
    commands:
      - mvn test
      - mvn checkstyle:checkstyle
      - echo "Installing JDK11 as its a dependency for sonarqube code analysis"
      - apt-get install -y openjdk-11-jdk
      - export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
      - mvn sonar:sonar -Dsonar.login=$LOGIN -Dsonar.host.url=$HOST -Dsonar.projectKey=$Project -Dsonar.organization=$Organization -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ -Dsonar.junit.reportsPath=target/surefire-reports/ -Dsonar.jacoco.reportsPath=target/jacoco.exec -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
      - sleep 5 
      - curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json 
      - cat result.json
      - if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi
```

### Step 6: Setup CodeBuild for Build Artifact:

- Buildspec file to build the artifact and store in code artifact

```
version: 0.2
env:
  parameter-store:
    CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN
phases:
  install:
    runtime-versions:
      java: corretto8
    commands:
      - cp ./settings.xml /root/.m2/settings.xml
  pre_build:
    commands:
      - apt-get update 
      - apt-get install -y jq 
      - wget http://www-eu.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
      - tar xzf apache-maven-3.5.4-bin.tar.gz
      - ln -s apache-maven-3.5.4 maven
  build:
    commands:
      - mvn clean install -DskipTests
artifacts:
  files:
    - target/**/*.war
  discard-paths: yes
```

### Step 7: Setup CodePipeline:

create pipeline 
source - code commit and repo name 
add build artifact stage 
create piepline 
add stage for sonarqube analysis
create bucket with folder 
add stage for deploy to s3
extact the artifact 
setup notification 
run the pipeline 