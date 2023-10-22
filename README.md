# project-14
## EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP
#### SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION
As part of the ongoing infrastructure development with Ansible started from Project 11, you will be tasked to create a pipeline that simulates continuous integration and delivery. Target end to end CI/CD pipeline is represented by the diagram below. It is important to know that both Tooling and TODO Web Applications are based on an interpreted (scripting) language (PHP). It means, it can be deployed directly onto a server and will work without compiling the code to a machine language.

The problem with that approach is, it would be difficult to package and version the software for different releases. And so, in this project, we will be using a different approach for releases, rather than downloading directly from git, we will be using Ansible uri module.
##### Set Up
This project is partly a continuation of your Ansible work, so simply add and subtract based on the new setup in this project. It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. This will be quite a lot of servers altogether (But you don’t have to create them all at once. Only create servers required for an environment you are working with at the moment. For example, when doing deployments for development, do not create servers for integration, pentest, or production yet).

Try to utilize your AWS free tier as much as you can, you can also register a new account if you have exhausted the current one. Alternatively, you can use Google Cloud (GCP) to rent virtual machines from this cloud service provider – you can get $300 credit here or here(NOTE: Please read instructions carefully to get your credits)

##### NOTE:
This is still NOT a cloud-focus project yet. AWS cloud end to end project begins from project-15. Therefore, do not worry about advanced AWS or GCP configuration. All we need here is virtual machines that can be accessed over SSH.

To minimize the cost of cloud servers, you don not have to create all the servers at once, simply spin up a minimal server set up as you progress through the project implementation and have reached a need for more.

To get started, we will focus on these environments initially.
- Ci
- Dev
- Pentest

Both SIT – For System Integration Testing and UAT – User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding our applications. But Pentest – For Penetration testing is where we will conduct security related tests, so some other tools and specific configurations will be needed. In some cases, it will also be used for Performance and Load testing. Otherwise, that can also be a separate environment on its own. It all depends on decisions made by the company and the team running the show.
What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools.
###### Ansible Inventory should look like this
├── ci

├── dev

├── pentest

├── pre-prod

├── prod

├── sit

└── uat
###### ci inventory file
[jenkins]
Jenkins-Private-IP-Address

[nginx]
Nginx-Private-IP-Address

[sonarqube]
SonarQube-Private-IP-Address

[artifact_repository]
Artifact_repository-Private-IP-Address
###### dev Inventory file
[tooling]
Tooling-Web-Server-Private-IP-Address

[todo]
Todo-Web-Server-Private-IP-Address

[nginx]
Nginx-Private-IP-Address

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
DB-Server-Private-IP-Address

###### pentest inventory file
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
Pentest-for-Todo-Private-IP-Address

[pentest-tooling]
Pentest-for-Tooling-Private-IP-Address
###### Observations:
- You will notice that in the pentest inventory file, we have introduced a new concept pentest:children This is because, we want to have a group called pentest which covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.
- The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up. Totally up to you how you want to do this. Whatever works for you is absolutely fine in this scenario.
This makes us to introduce another Ansible concept called group_vars. With group vars, we can declare and set variables for each group of servers created in the inventory file.

For example, If there are variables we need to be common between both pentest-todo and pentest-tooling, rather than setting these variables in many places, we can simply use the group_vars for pentest. Since in the inventory file it has been created as pentest:children Ansible recognizes this and simply applies that variable to both children.
#### ANSIBLE ROLES FOR CI ENVIRONMENT
###### Why do we need SonarQube?
SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities. Watch a short description here. There is a lot more hands on work ahead with SonarQube and Jenkins. So, the purpose of SonarQube will be clearer to you very soon.
###### Why do we need Artifactory?
Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of your build process is stored. It can be used for certain other automation, but we will it strictly to manage our build artifacts.
###### Configuring Ansible For Jenkins Deployment
In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,
- Navigate to Jenkins URL
- Install & Open Blue Ocean Jenkins Plugin
- Create a new pipeline
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9512ae21-f934-43a0-bf34-ade3bff52cfb)

Select GitHub > Connect Jenkins with GitHub > Login to GitHub & Generate an Access Tokenhttps://www.dareyio.com/wp-content/uploads/2021/07/Jenkins-Create-Access-Token-To-Github.png > Copy Access Token > Create a new Pipeline > 

At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/39952b29-a544-438c-b028-83289ca2cdbe)

#### Let us create our Jenkinsfile
Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.
Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/28013634-75f0-4697-b347-ce253b9c58fa)

- Now go back into the Ansible pipeline in Jenkins, and select configure
- Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/d6428401-434b-4403-94e6-186bce185d5d)

- Back to the pipeline again, this time click "Build now"
- This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.
- To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.
  Click on Blue Ocean
- To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.
- Click on the "Administration" button
- Navigate to the Ansible project and click on "Scan repository now"
- Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.
- In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.
    ![](https://github.com/UzonduEgbombah/project-14/assets/137091610/40442587-7bcf-4023-8781-1eb2dbe4004f)

# A QUICK TASK FOR YOU!
1. Create a pull request to merge the latest code into the main branch
2. After merging the PR, go back into your terminal and switch into the main branch.
3. Pull the latest change.
4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
- Package 
- Deploy 
- Clean up
5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
6. Eventually, your main branch should have a successful pipeline like this in blue ocean

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/926b6b31-e14d-4830-9019-b2b7233b9fa7)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/0b6da599-3290-4ff1-96f7-66c6789de315)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/0fb6a8d2-de06-4724-910d-74f5fdc56f76)

## RUNNING ANSIBLE PLAYBOOK FROM JENKINS
Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

- Installing Ansible on Jenkins
- Installing Ansible plugin in Jenkins UI
Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)
###### Note:
Ensure that Ansible runs against the Dev environment successfully.
- Possible errors to watch out for:
- Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)
- Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.
- https://wiki.jenkins.io/display/JENKINS/Building+a+software+project
#### Possible issues to watch out for when you implement this
Remember that ansible.cfg must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.

If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.

Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.

If everything goes well for you, it means, the Dev environment has an up-to-date configuration. But what if we need to deploy to other environments?

Are we going to manually update the Jenkinsfile to point inventory to those environments? such as sit, uat, pentest, etc.
Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.
Think about those for a minute and try to work out which one sounds more like a better solution.

Manually updating the Jenkinsfile is definitely not an option. And that should be obvious to you at this point. Because we try to automate things as much as possible.

Well, unfortunately, we will not be doing any of the highlighted options. What we will be doing is to parameterise the deployment. So that at the point of execution, the appropriate values are applied.
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/d49aba6c-feef-4c16-83db-52f4306b2ce3)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/5371de60-eecc-48d3-b522-dd5d4489891a)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/ecb5a257-8cd2-42f5-9c0e-41d5402b4d27)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/b5123042-a573-4d93-a23e-aa2669f2598b)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/02cbdf0c-8946-47a3-9b1f-90801e7ddb1d)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/be8a10d0-4be1-4a45-82a9-0ced5a11e92d)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/92c4af43-f96e-4f4d-8869-47a4d39bee27)

# Parameterizing Jenkinsfile For Ansible Deployment
To deploy to other environments, we will need to use parameters.

Update sit inventory with new servers
[tooling]
SIT-Tooling-Web-Server-Private-IP-Address
[todo]
SIT-Todo-Web-Server-Private-IP-Address
[nginx]
SIT-Nginx-Private-IP-Address
[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python
[db]
SIT-DB-Server-Private-IP-Address
Update Jenkinsfile to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/0cf70d7e-992c-4305-97ed-618d6581227e)

In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}
From now on, each time you hit on execute, it will expect an input.
![Screenshot 2023-10-13 121511](https://github.com/UzonduEgbombah/project-14/assets/137091610/c729a579-2e88-47d3-a38e-fb2f18a070e3)

# CI/CD PIPELINE FOR TODO APPLICATION
We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04

#### Phase 1
– Prepare Jenkins
Fork the repository below into your GitHub account
https://github.com/darey-devops/php-todo.git
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/54cfb186-4c3a-4912-bed1-fdcfb119bdf3)
On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/351c6d53-1682-4a9e-b2f3-02dd23727551)


![](https://github.com/UzonduEgbombah/project-14/assets/137091610/ed7b2579-6615-4168-8b09-3f4e81d25c4f)

Install Jenkins plugins
Plot plugin
Artifactory plugin
We will use plot plugin to display tests reports, and code coverage information.
The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
In Jenkins UI configure Artifactory
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/014de982-dcce-4d1f-a54e-d55001d9594f)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/cb2ab997-e177-43e1-a362-fa2d873ec91d)

- In Jenkins UI configure Artifactory
- Configure the server ID, URL and Credentials, run Test Connection
##### Phase 2
- Integrate Artifactory repository with Jenkins
- Create a dummy Jenkinsfile in the repository
- Using Blue Ocean, create a multibranch Jenkins pipeline
- On the database server, create database and user
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
Update the database connectivity requirements in the file .env.sample
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/e3e11080-f211-4127-8fa4-84af55db5b2e)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/71113874-4192-4f2d-ad4b-2bac50f927ca)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/c5300472-d753-4bde-8239-bab638b9a40c)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/cda38043-c387-4516-923a-3fead3af430d)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/85359e03-55a4-40d7-832f-4b54046a96d3)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/774cad85-a2e8-4110-a786-d3001b5a9a1f)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/28556d65-67d6-4277-876d-54ddaa5da9d8)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/472423a1-c9f6-44a4-bdea-83f53d7d130a)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/73bffea3-2725-41ca-a104-4ee0a23c0489)

Update Jenkinsfile with proper pipeline configuration
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/709227f6-6fe8-4f57-a178-4a045ac58faa)

#### Notice the Prepare Dependencies section

The required file by PHP is .env so we are renaming .env.sample to .env
Composer is used by PHP to install all the dependent libraries used by the application

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/1ab34ed1-f241-4569-8ca7-98f9245bca28)

php artisan uses the .env file to setup the required database objects – (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/dd654f0d-deda-48a9-8de2-0bb107c5a5b6)

Update the Jenkinsfile to include Unit tests step
![Screenshot 2023-10-18 205312](https://github.com/UzonduEgbombah/project-14/assets/137091610/4823cb1e-42d4-4b29-a98c-b9f4adacc2e2)

#### Phase 3
###### Code Quality Analysis
This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. As a DevOps engineer, you also have a role to play. Especially when it comes to setting up the tools.
For PHP the most commonly tool used for code quality analysis is phploc.
The data produced by phploc can be ploted onto graphs in Jenkins.
Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/e435454a-ba67-45eb-9508-6495eca85f89)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9f217375-9209-4549-8bd5-af90ca727acc)

##### Plot the data using plot Jenkins plugin.
This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots’ data series latest values are pulled from the CSV file generated by phploc.

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/da2dc763-86d0-475d-a551-3cf8f788930e)

You should now see a Plot menu item on the left menu. Click on it to see the charts. (The analytics may not mean much to you as it is meant to be read by developers. So, you need not worry much about it – this is just to give you an idea of the real-world implementation).

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9a602e67-740d-42f8-b787-60d8edeb0254)

- Bundle the application code for into an artifact (archived package) upload to Artifactory
  
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/5aa46b0e-4a4d-433f-8745-2a0895143759)

- Publish the resulted artifact into Artifactory
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/22974cf7-e2d8-4081-8aca-7803f2667cc3)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/6df7e5b5-c1d2-4b43-a49d-4cdef17700b7)

- Deploy the application to the dev environment by launching Ansible pipeline

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/bce6a816-4fe7-478e-b791-7d4f83843062)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/1f157634-a6e7-4791-9df4-65b7671beebd)

The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch. Hence, we have ansible-project/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is env and its value is dev. Meaning, deploy to the Development environment.

But how are we certain that the code being deployed has the quality that meets corporate and customer requirements? Even though we have implemented Unit Tests and Code Coverage Analysis with phpunit and phploc, we still need to implement Quality Gate to ensure that ONLY code with the required code coverage, and other quality standards make it through to the environments.

To achieve this, we need to configure SonarQube – An open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

#### SONARQUBE INSTALLATION
Before we start getting hands on with SonarQube configuration, it is incredibly important to understand a few concepts:

Software Quality – The degree to which a software component, system or process meets specified requirements based on user needs and expectations.
Software Quality Gates – Quality gates are basically acceptance criteria which are usually presented as a set of predefined quality criteria that a software development project must meet in order to proceed from one stage of its lifecycle to the next one.
SonarQube is a tool that can be used to create quality gates for software projects, and the ultimate goal is to be able to ship only quality software code.

Despite that DevOps CI/CD pipeline helps with fast software delivery, it is of the same importance to ensure the quality of such delivery. Hence, we will need SonarQube to set up Quality gates. In this project we will use predefined Quality Gates (also known as The Sonar Way). Software testers and developers would normally work with project leads and architects to create custom quality gates.

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/b71f9126-1f6b-42d5-b006-575c3993e947)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/583a3f67-852c-4d39-a199-e892e1ac795a)
## Install and Setup PostgreSQL 10 Database for SonarQube
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/8e4b1ee8-7c51-48c2-b825-693ee355a211)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/6efce00e-03d8-4249-94e2-6be541a435ac)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/bc288158-59bf-4fae-852d-a99f810200e5)

## CONFIGURE SONARQUBE
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/bc20735b-a75b-48ad-97bc-98aac6194045)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/a9f29d4c-e8e3-43f3-9ab6-3db78dba6181)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/a6642422-a296-4faa-b6f7-463c1ef61d6a)

#### Access SonarQube
To access SonarQube using browser, type server’s IP address followed by port 9000
http://server_IP:9000 OR http://localhost:9000
Login to SonarQube with default administrator username and password – admin

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/7dacff52-dbcc-425c-803b-b27c80aea9bb)

## CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE
In Jenkins, install SonarScanner plugin
Navigate to configure system in Jenkins. Add SonarQube server as shown below:
Manage Jenkins > Configure System
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9a1f03a6-02f0-495f-97bc-68b1d3e3c15f)

#### Generate authentication token in SonarQube
User > My Account > Security > Generate Tokens
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/d2dad84f-0cd9-40a3-b652-598b38779b53)

Configure Quality Gate Jenkins Webhook in SonarQube – The URL should point to your Jenkins server http://{JENKINS_HOST}/sonarqube-webhook/

Administration > Configuration > Webhooks > Create

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/f557b6e2-740d-4f64-83db-a380c9776021)

Setup SonarQube scanner from Jenkins – Global Tool Configuration

Manage Jenkins > Global Tool Configuration   >>>

- Update Jenkins Pipeline to include SonarQube scanning and Quality Gate
  Below is the snippet for a Quality Gate stage in Jenkinsfile.
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/f23cdb45-95a6-48a0-8cf0-d57190066c6f)

#### NOTE:
The above step will fail because we have not updated `sonar-scanner.properties
- Configure sonar-scanner.properties – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to 
  configure the properties file in which SonarQube will require to function during pipeline execution.
  
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/81e838d6-9357-4051-8e5b-4a6c16a27a53)

###### Open sonar-scanner.properties file
sudo vi sonar-scanner.properties

###### Add configuration related to php-todo project
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/5c6ed529-cfae-4194-9d44-ba3af15674b9)

###### End-to-End Pipeline Overview
Indeed, this has been one of the longest projects from Project 1, and if everything has worked out for you so far, you should have a view like below:
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/8cb69419-f600-432e-b894-d753b123c15c)

But we are not completely done yet!

The quality gate we just included has no effect. Why? Well, because if you go to the SonarQube UI, you will realise that we just pushed a poor-quality code onto the development environment.

- Navigate to php-todo project in SonarQube

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/04fae099-6f8d-46ca-af32-90a276218932)


There are bugs, and there is 0.0% code coverage. (code coverage is a percentage of unit tests added by developers to test functions and objects in the code)

If you click on php-todo project for further analysis, you will see that there is 6 hours’ worth of technical debt, code smells and security issues in the code.
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9a8b7ad9-32c8-42eb-834c-982fdca7147d)

In the development environment, this is acceptable as developers will need to keep iterating over their code towards perfection. But as a DevOps engineer working on the pipeline, we must ensure that the quality gate step causes the pipeline to fail if the conditions for quality are not met.
###### Conditionally deploy to higher environments
In the real world, developers will work on feature branch in a repository (e.g., GitHub or GitLab). There are other branches that will be used differently to control how software releases are done. You will see such branches as:

- Develop
- Master or Main
(The * is a place holder for a version number, Jira Ticket name or some description. It can be something like Release-1.0.0)
- Feature/*
- Release/*
- Hotfix/*
  etc.

There is a very wide discussion around release strategy, and git branching strategies which in recent years are considered under what is known as GitFlow (Have a read and keep as a bookmark – it is a possible candidate for an interview discussion, so take it seriously!)

Assuming a basic gitflow implementation restricts only the develop branch to deploy code to Integration environment like sit.

Let us update our Jenkinsfile to implement this:

 - First, we will include a When condition to run Quality Gate whenever the running branch is either develop, hotfix, release, main, or master
 - Then we add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable
 The complete stage will now look like this:

    stage('SonarQube Quality Gate') {
      when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
            }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }

To test, create different branches and push to GitHub. You will realise that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

If everything goes well, you should be able to see something like this:

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/5b212fae-59ff-4639-b36c-d51a6b1b9574)

Notice that with the current state of the code, it cannot be deployed to Integration environments due to its quality. In the real world, DevOps engineers will push this back to developers to work on the code further, based on SonarQube quality report. Once everything is good with code quality, the pipeline will pass and proceed with sipping the codes further to a higher environment.
#### Complete the following tasks to finish Project 14

Introduce Jenkins agents/slaves – Add 2 more servers to be used as Jenkins slave. Configure Jenkins to run its pipeline jobs randomly on any available slave nodes.

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/cd2a1a84-48f4-4336-86a9-85c72b798169)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/55d1028e-3cdc-46e5-8dfb-9848376eca6c)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/9e111aa3-da1c-4a2b-a2b6-8a2876b808ba)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/be10239e-8de7-406e-9f37-17620c5bb3ff)

Configure webhook between Jenkins and GitHub to automatically run the pipeline when there is a code push.
![](https://github.com/UzonduEgbombah/project-14/assets/137091610/dff3f7c7-0661-4592-aabb-bc5624c690ae)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/4fc468ba-b9c2-41aa-8fec-376dbd389306)

![](https://github.com/UzonduEgbombah/project-14/assets/137091610/250f26e2-619e-40c3-813a-f20f8a59095a)





































































