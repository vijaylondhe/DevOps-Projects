# Project-07: Continuous Delivery of Java Application with Jenkins & Tools 

### Continuous Delivery Process:

- In Agile SDLC, developer makes regular code changes (commits/pull requests) and this changes need to be build and tested, also the build artifacts and test reports are generated. 

- After the artifacts are generated and before the application is deployed to the production systems, it should pass from the various integraion and performance tests.

- This testing is performed on the TEST/PRE-PROD/STAGING environments.

- Afte the test reports get evaulated, manual approval is given for the application depoyment on production environment.


### Objectives:

Setup the Continuous delivery pipeline with jenkins to fetch the latest docker image of our application and deploy docker container on ECS.

### Tools/Services Used:

- `Jenkins`: Continuous Integration and Continuous Delivery Server.

- `Nexus Sonatype Repository`: Download Maven Dependency and Upload Build Artifact.

- `Sonarqube`: Sonarqube scanner for code analysis, and sonarqube server to check the quality gates.

- `Maven`: Used in jenkins to build the java application.

- `Git`: Used to store our source code with jenkins pipeline code.

- `Slack`: For notification 

- `Docker`: To build the docker image from our application

- `AWS Elastic Container Registry (ECR)`: To publish and store the docker image.

- `AWS Elastic Container Service (ECS)`: Used to host the docker containers for our application.

- `AWS CLI`: Used to run from the jenkins to fetch the latest image from ECR and and run the service in ECS.


### Architecture:

### Flow of Execution:

