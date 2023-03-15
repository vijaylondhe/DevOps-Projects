# Project-11: Continuos Deilvery and Configuration Management with Jenkins & Ansible 

### Objective:

- Continuos Integration: Build, test and deploy the artifact to nexus repository 
- Continuos Delivery: Using ansible download the artifact from the nexus repository and deploy it to the tomcat server in staging and production environment.

### Tools & Services Used:
- Jenkins 
- Nexus Sonatype Repository
- Sonarqube 
- Maven
- Git 
- Slack
- AWS EC2
- Tomcat
- Ansible 

### Architecture:

![GitHub Light](./snaps/pro-11-cicd-ansible1.jpg)

### Flow of Execution:

1. Update the github webhook and new jenkins ip and resize the jenkins instance volume 
2. Launch the new instance for app01-staging 
3. Jenkins Prerequisite
   - Install Ansible 
   - Install Ansible Plugin 
   - Create Credentials for app01-staging instance 
   - Timestamp varibale 
4. Create separate branch for the pipeline in source code (Gitrepo)
   - Copy ansible code from vprofile-project repository to our own repository
5. Create DNS record in Route53 for app01-staging
6. Create inventory file in ansible code  
7. Update the security group rules
   - Allow port 22 in App SG from Jenkins SG
   - Allow port 8081 in Nexus SG from App SG   
8. Write Jenkinsfile to run Ansible Playbook from the Jenkins 
9. Create Pipeline in Jenkins and test it 
10. Update Ansible code in prod branch
11. Launch app01-prod, dns record in route53, create inventory file
12. Create Pipeline in Jenkins and test it
13. Use git merge to promote the changes to prod branch
