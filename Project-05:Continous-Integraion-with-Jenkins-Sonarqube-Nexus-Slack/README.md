# Project-05: Continuous Integration with Jenkins, Sonarqube, Nexus and Slack

### Objectives:

- Build and Test the source code for every commit made by developer. 
- Setup completely automated process.
- Notify to developer for every build status.
- Fix the code if error or bug found instantly rather than waiting.
- Fault isolation.
- Short MTTR (Mean Time to Repair).
- Fast turn around on new feature changes.
- Less dispruptive.

### Tools Used:

- Jenkins: Continuous integration server
- Git/GitHub: Version control system
- Maven: To build the code 
- Checkstyle: To analysis the code
- Slack: For the notification
- Sonatype Nexus: To store the artifact and download the dependency for maven 
- Sonarqube: To analysis the code using sonar scanner and publish the report to dashboard
- AWS EC2: Compute resource to host all this tools


### CI Workflow:

![GitHub Light](./snaps/ci_workflow_updated.jpg)

### Flow of Execution:

- Login to AWS
- Create key pair
- Create Security group for Jenkins, Sonarqube and Nexus
- Create EC2 instance with user data for Jenkins, Sonarqube and Nexus
- Post Installation
  - Jenkins setup and plugins
  - Nexus setup and repository setup 
  - Sonarqube login test
- Git
  - Create github repository & migrate code 
  - Integrate github repo with vscode and test it
- Build job with nexus integration 
- Setup Github Webhook
- Sonarqube server integration
- Nexus artifact upload stage
- Slack notification


### Architecture:

![GitHub Light](./snaps/pro-05-ci-architecture_updated.jpg)


### Steps:

#### Step 1: Create Key Pair and Security Group:

- Login to AWS account
- Go to the EC2 service
- Select N. Virginia as region
- Click on Key Pair
- Key Name: vprofile-ci-key
- Key Pair Type: RSA
- Private key file format: .pem 
- Click on Create Key Pair

![GitHub Light](./snaps/<>.jpg)

- Go to VPC service
- Click on Security Groups
- Create Security Group for Jenkins Instance
  - Security Group Name: jenkins-sg
  - Description: Security Group for Jenkins Instance
  - VPC: default 
  - Inbound rules: 
    - Rule 1
    ```
    Type: Custom TCP 
    Port range: 22
    Source: MyIP
    Description: for ssh access to jenkins 
    ```

    - Rule 2
    ```
    Type: Custom TCP 
    Port range: 8080
    Source: Anywhere IPv4 (0.0.0.0/0)
    Description: for jenkins console access and for github webhook
    ```

    - Rule 3
    ```
    Type: Custom TCP 
    Port range: 8080
    Source: Anywhere IPv6 (::/0)
    Description: for jenkins console access
    ```
  
  - Click on Create Security Group 

- Create Security Group for Nexus Instance
  - Security Group Name: nexus-sg
  - Description: Security Group for Nexus Instance
  - VPC: default 
  - Inbound rules: 
    - Rule 1
    ```
    Type: Custom TCP 
    Port range: 22
    Source: MyIP
    Description: for ssh access to nexus instance 
    ```

    - Rule 2
    ```
    Type: Custom TCP 
    Port range: 8081
    Source: MyIP
    Description: for nexus console access 
    ```

    - Rule 3
    ```
    Type: Custom TCP 
    Port range: 8081
    Source: Custom (sg id of jenkins-sg)
    Description: to upload artifact to nexus from jenkins job and download the dependency from nexus
    ```
  
  - Click on Create Security Group 

- Create Security Group for Sonar Instance
  - Security Group Name: sonar-sg
  - Description: Security Group for Sonar Instance
  - VPC: default 
  - Inbound rules: 
    - Rule 1
    ```
    Type: Custom TCP 
    Port range: 22
    Source: MyIP
    Description: for ssh access to sonar instance 
    ```

    - Rule 2
    ```
    Type: Custom TCP 
    Port range: 80
    Source: MyIP
    Description: for sonar console access 
    ```

    - Rule 3
    ```
    Type: Custom TCP 
    Port range: 80
    Source: Custom (sg id of jenkins-sg)
    Description: to upload test results to sonar from jenkins job 
    ```
  
  - Click on Create Security Group 

- Modify Security Group of Jenkins Instance
  - Inbound rules:
    - Rule 4
    ```
    Type: Custom TCP 
    Port range: 8080
    Source: Custom (sg id of sonar-sg)
    Description: for ssh access to soanr instance 
    ```