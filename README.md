# 3-tier-devsecops
created 2 mns
connected to local
installed jhenkings in first one
installed docker in 2nd one but it failed and connected to command prompt
Day 1:
Expalining what we are going to do in this session
Day 2:
deploying application in local
Day 3:
Setting up infra for building CI Pipeline inside which we write diff stages including diff security tools
Infra needed
============
1. Github repo 2. Jenkins(CICD) 3. SonarQube 4. Trivy  5. Docker (We will build docker images)

SonarQube : SAS Security tool to make sure our code is not having any kind of bugs or any kind of code realted issue
Trivy : Another security tool to make sure that there are no vulnerabilities within the dependencies what we use in code
        Example: we are using NOde.js based project inside API folder which is backend and inside our client folder we have package.json file in both backend and front end this file contains dependencies which we utilize in our project with specific versions. if we are having dependencies with versions we should check if version is proper or having any kind of vulnerabilites. this can done by TRIVY

Stages inside our pipeline
==========================
when we talk about CI/CD important aspect is that we can automate it, so that very less human effort is required to run pipeline
Example: Client says I want to add some new features to project like i want to change background colour of project. 
        Then he will raise ticket with the information he want to change, once ticket is raised it will be assigned to developer to work on specific feature for cahnaging background colour.
        Developer will not directly change the code and merge it to main branch, they will test first locally this can be done by below assumption.
        let us assume we are having two barnches main and dev branches
        we will make changes in dev branch from dev branch  we will create feature branch with name feature/background change.
        developer will clone this branch to their local and they will write the code for changes and test it  by running app locally. if no errors found they will push code to feature branch in github and merge that branch bak to dev branch.
        once it is merged it will trigger the CI Pipeline(here comes automation, pipeline should trigger when merging happens). Devops engineer is responsible for this automation, for this we have to setup concept known as WEBHOOK. In webhook we will define as soon as any changes happen to specific branches of repository it should trigger a specific pipeline. Till Branching and merging its developers work, writing webhook and writing CI Pipeline will be devops engineer work

Stages:
-------
1. Git Checkout 2. Compilation 3. Gitleaks 4. FS Trivy 5. Unit Testing 6. SonarQube Analysis 7. Quality gate check 8. Docker Image 9. FS Trivy 10. Push Image to DockerHub Repo or other repo

Git Checkout It will create local copy of source code inside jenkins.
Compilation : we execute a command to find any kind of syntax based error ex: missed comma or bracket or spelling mistakes.
Gitleaks : Ex: in you source code you added some sensitive data like pwds, API Keys, Tokens etc. which should not be added, then 
            gitleaks will find such info and tells in which line its added which you ahve to remove.It acts as security tool to check where secrets are added.
FS Trivy : File Scan by Trivy which will check dependency file. Ex: Java dependency file is com.xml where we have info about
           dependencies, in Node.js we have package.json, in Dotnet we have .proj file, in python we have requirements.txt, so if we execute project based application trivy file system will scan this file and gets info about dependencies we are using in our code. After that it will compare that info about versions with a database which tells which version is vulnerable and not good to use and that will be stored in a report, so we can update our dependencies based on that report, and make sure our app is using secure dependencies and is not vulnerable.
Unit Testing : Unit test cases written will be executed. we dont have to write UT test cases but wwe can execute them directly in
               our pipeline.
SonarQube : It will go through line by line complete code that we have in our repository, and will find out any kind of bugs 
            available inside the code like any logic is wrong which is making application behave in a wrong way. Vulnerability means the section of code that is vulnerable to attack. Ex: you are writing authentication code and you have given permission to viewer role also and admin role also this is vulnerability kind of thing.  So sonarqube will find those kind of bugs and tell us the severity like its critical or minor like that. second it will tell in which exact file and which line issue there. third it will tell how to solve it. if using sonarqube you can make sure code is bug free not having any kind of code related issue which will cause any kind of issue in future. It will also find Code Coverage(how much % of code is covered in Unit test cases)  also.
Quality Gatecheck : Ex: after Sonarqube analysis you found 5 bugs and 1 vulnerability and 75% code coverage in sonarqube report. 
            Each company will have their own treshold as 0 bugs 0 vulnerabilites min 80% Code coverage. which means your code is in compilance state when you meet this threshold. This Code Compilance check is done by Quality Gatecheck step.
Docker Image : We will build docker image and tag it with specific version. We will Build Docker image just to make application 
            portable. Means when we bulid image we will get a PAckage. That package is known as Docker image. This Package is combined or integrated with everything that is required to run the application. Once the package is created you can run your application in any platform irrespective of kind of paltform. This can be done by Docker.
            Docker makes our application portable so that we can share with anyone who can run it.
FS Trivy : we will scan docker image file with trivy to find vulnerabilities
Push image: we can easily pull images from rerpos and publish using kubernetes
----****----
Main Purpose of CI Pipeline is to make sure app is secureabd it is ready for deployment and should be automated completely.
CD Pipeline we will deploy the application which got ready in CI Pipeline.
Infra Setup
===========
1. Create 2 VMS in AWS for Jenkins and SonarQube with T2Medium as instance and keep ports 3000-9000,443,587,80,22 open
2. Connect to VMS using Mobaxtream, after connecting use below commands
   sudo apt update --- will update internal pkgs to latest versions
   install java using web commands
   install jenkins after java using commands in web once it is installed connect to it using web
   copy ip add of VM and :8080, after connecting install plugins in jhenkins
      
   in sonarQube VM
   sudo apt update
   install docker - sudo apt install docker.io -- when you install docker only root user has permissions to execute docker cmnd to add ubuntu user to run docker cmnds use below cmnd. when you install docker by default docker grp will be created called docker. whichever user is added to taht grp he will be having permissions to execute docker cmnds.
   sudo usermod -aG docker ubuntu -- adding ubuntu user to docker grp
   newgrp docker -- to apply these changes immediately, either logout and login or use this cmnd
   docker run -d --name sonar -p 9000:9000 sonarqube:lts-community 
   -d-- running in dedashed mode, -p(port mapping) 1st 9000(host port i.e port on VM) 2nd 9000(conatiner port. The port on which SonarQube will be running inside docker conatiner) then docker image name
   docker ps --to check if container is running or not
   login to sonarqube web using ip add and :9000  Default usename and pwd for sonarqube will be admin admin respectively

3. Then login to jenkins --> Manage Plugins --> plugins --> Available plugins --> srch stage View and install that plugin
   New --> CI(Name of job)--> pipeline(type of job)-->ok--> discard old builds--> max 3 --> scrool down in script section write jenkins code --> new to groovy try sample helloworld by clicking in right corner of script section--> 
   syntax explaination----
   pipeline -- block inside which complete thing will be written
   agent any -- where this pipeline will be running
   stages -- block where all stages will be written
   copy sample stages and create multiple stages, if you dont know syntax you can use pipeline syntax at stage down.
   pipeline syntax --> expand sample step --> git provide comnds and copy o/p syntax paste in code
   to install Node.js inside pipeline 2 options 1. directly on jenkins server or 2. using a plugin
   1. use nodejs based cmd to install it. 2. istall plugin and configure in code
   when you directly install in server you dont have to define it anywhere in pipeline
   installed via plugin you have to define it in plugin
   install node.js plugin and to configure navigate to tools --> Node.js installation add version --> apply
   to define in pipeline aftre plugins use tools section in code
   Tools section does not support all kind of tools like maven jdk nodejs can be defined here but not all at once
   install gitleaks in vm using sudo apt install gitleaks and use dorectly cmnds in code
   After gitleaks check if code is proper or not by running pipeline once. if it is successful add trivy and sonarqube stage
   too work with sonarqube few setings needs to be done in jenkins need 2 things 1.sonarqube scanner 2.SonarQube server
   Scanner will be running inside jenkins performing the analysis generating the report and publishing the report over the server
   Server is already steup we have to setup scanner in jenking for that navigate to Manage jenkins --> plugins --> available plugins --> srch sonarqube scanner and install and configure in tools give name and select version of scanner and click apply, next go to system find sonarQube server section provide server URL and server authentiacation token, to generate token go to sonarqube Server --> Administrator --> Security --> User --> Click on Tokens provide any name to token and generate --> Copy that token and go to jenkins --> Credentials --> Global --> Add Credentials --> Kind Secret text --> Pate token in secret tab --> Provide any id and description --> click on create after that inside jenkins tell where to publish scanner report becoz jenkins don't know where to publish it by default.
   for that provide URL of server and credentials to access it


   // Jenkinsfile
pipeline {
  agent any
  tools {    ----format tool name tool type----
    nodejs 'nodejs23'
  }
  stages {
    stage('Git Checkout') {          ----from where should code be picked----
      steps {
        git branch: 'dev' , url: 'https://github.com/Praveena1931/3-tier-devsecops.git'
      }
    }

    stage('frontend compilation') {
      steps {
        dir('client') {
            sh 'find . -name "*.js" -exec node --check {} +' ---checks all .js files for compilation in batch---
        }
        
      }
    }

    stage('backend compilation') {
      steps {
        dir('api') {
            sh 'find . -name "*.js" -exec node --check {} +' ---checks all .js files for compilation in batch---
        }
        
      }
    }

    stage('gitleaks scan') {
      steps {
        sh 'gitleaks detect --source ./client --exit-code=1'
        sh 'gitleaks detect --source ./api --exit-code=1'
      }
    }

    stage('Hello') {
      steps {
        echo 'Hello, World!'
      }
    }

    stage('Hello') {
      steps {
        echo 'Hello, World!'
      }
    }
  }
}






 
