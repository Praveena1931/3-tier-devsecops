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

-Git Checkout: It will create local copy of source code inside jenkins.
-Compilation : we execute a command to find any kind of syntax based error ex: missed comma or bracket or spelling mistakes.
-Gitleaks : Ex: in you source code you added some sensitive data like pwds, API Keys, Tokens etc. which should not be added, then 
            gitleaks will find such info and tells in which line its added which you ahve to remove.It acts as security tool to check where secrets are added.
-FS Trivy : File Scan by Trivy which will check dependency file. Ex: Java dependency file is com.xml where we have info about
           dependencies, in Node.js we have package.json, in Dotnet we have .proj file, in python we have requirements.txt, so if we execute project based application trivy file system will scan this file and gets info about dependencies we are using in our code. After that it will compare that info about versions with a database which tells which version is vulnerable and not good to use and that will be stored in a report, so we can update our dependencies based on that report, and make sure our app is using secure dependencies and is not vulnerable.
-Unit Testing : Unit test cases written will be executed. we dont have to write UT test cases but wwe can execute them directly in
               our pipeline.
-SonarQube : It will go through line by line complete code that we have in our repository, and will find out any kind of bugs 
            available inside the code like any logic is wrong which is making application behave in a wrong way. Vulnerability means the section of code that is vulnerable to attack. Ex: you are writing authentication code and you have given permission to viewer role also and admin role also this is vulnerability kind of thing.  So sonarqube will find those kind of bugs and tell us the severity like its critical or minor like that. second it will tell in which exact file and which line issue there. third it will tell how to solve it. if using sonarqube you can make sure code is bug free not having any kind of code related issue which will cause any kind of issue in future. It will also find Code Coverage(how much % of code is covered in Unit test cases)  also.
-Quality Gatecheck : Ex: after Sonarqube analysis you found 5 bugs and 1 vulnerability and 75% code coverage in sonarqube report. 
            Each company will have their own treshold as 0 bugs 0 vulnerabilites min 80% Code coverage. which means your code is in compilance state when you meet this threshold. This Code Compilance check is done by Quality Gatecheck step.
-Docker Image : We will build docker image and tag it with specific version. We will Build Docker image just to make application 
            portable. Means when we bulid image we will get a PAckage. That package is known as Docker image. This Package is combined or integrated with everything that is required to run the application. Once the package is created you can run your application in any platform irrespective of kind of paltform. This can be done by Docker.
            Docker makes our application portable so that we can share with anyone who can run it.
-FS Trivy : we will scan docker image file with trivy to find vulnerabilities
-Push image: we can easily pull images from rerpos and publish using kubernetes
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
   Server is already steup we have to setup scanner in jenking for that navigate to Manage jenkins --> plugins --> available plugins --> srch sonarqube scanner and install and configure in tools give name and select version of scanner and click apply, next go to system find sonarQube server section provide server URL and server authentiacation token, to generate token go to sonarqube Server --> Administrator --> Security --> User --> Click on Tokens provide any name to token and generate --> Copy that token and go to jenkins --> Credentials --> Global --> Add Credentials --> Kind Secret text --> Pate token in secret tab --> Provide any id and description --> click on create after that inside jenkins tell where to publish scanner report becoz jenkins don't know where to publish it by default.for that provide URL of server and credentials to access it
   ex. when you want to open xyz file you will provide path like home/bin/xyz same like this cmnd $SCANNER_HOME/bin/sonar-scanner
   for qualitygate check to pass successfully in sonarqube server configure webhook server--> administration --> configuration --> webhook --> create --> provide name --> jenkins url provide --> Add
   we need to add webhook becoz, we are sending report from jenkins to sonarqube, in qualitygate check it will compare generated report with existing report for thst we have to generate webhook.
   There is no direct plugin available for trivy inside jenkins so we have to install manually using google
   srch for trivy instllation steps and install on VM, as we are directly installing we don't need to define in pipeline 
   
   if you didnt find pipeline sysntax by default restart jenkins by typing rester after URL

   // Jenkinsfile
pipeline {
  agent any
  tools {    ----format tool name tool type----
    nodejs 'nodejs23'
  }
  environment{
    SCANNER_HOME = tool 'sonar-scanner' "scanner name where we provided token details in credentials"
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

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar')"this refers to name we provided in system confuguration tab"{
          sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName = NodeJS-Project \
               -Dsonar.projectKey  = Nodejs-project''' 
        }
      }
    }

    stage('Quality Gate Check') { timetout to setup time of run
      steps {
        timeout(time: 1, unit: 'HOURS'){
            waitForQualityGate abortPipeline: false, credenatialsId: 'sonar-token'
        }
      }
    }

    stage('Trivy FS Scan') {
      steps {
          sh 'trivy fs -format table -o fs-report.html .'
      }
    }
  }
}

Day 4:
Dockarizing application to run on any platform, multistage docker file, docker compose file.
why docker? when we ran our app locally we have to install Nodejs, mySQL server, user, db should be setup, all the dependencies should be installed like NPM, backend should be started manually, frontend should be strted manually line npmstart, 5000,3306,3000 ports should also be open. these are all prerequisite. if you share application to anyone they have to install everything from scratch and theymight not have any knowledge on them. What if I package all these things(prerequisites) along with application, like source code, i will package nodejs, i will package SQL server, commands to run app, and package specific ports also. So I can create this new kind of package which has all information available plus required tools to run project, and also commands to run project, That means i can share this kind of application and tell to someone that instead of setting up things manually one by one you can trigger or start this package.
so when he run specific command singlecmnd for running package automatically a box will be created which will be called as **container.** Container is an isolated environment inside which your application will run without any issues. So automatically a container will be created in side which our application will run.
As comapred to local run this is much easier where we need one cmnd to run new package. Now this package becomes the portable application. So making the application portable packing the dependencies and other information replated to application and shipping that package to anyone, this whole process is known as **dockarizing** and tool used for doing this is **Docker**. Anyone who wants to run this package application need to have only one application that is DOCKER. In DOcker each line is layer

Steps
=====
1. Dockarizing Backend 2. Dockarizing Frontend 3. Connect both frontend and backend together along with MYSQL for this we need **Docker compose**.
If Your app is having only one component like only frontend you can create an Image and run it, what if you have multiple components line backend frontend database, we have to create 3 images i.e 3 containers will be created for each we need to make sure these containers are created one by one or create another file docker compose in this we define all the information like for all three components create docker images and create Ccontainers and connect them together. After that run Docker compose command which will automatically going to cerate containers make sure they are connected and start the app.

MultiStage Docker file:
=========
Ex: Building Frontend single stage
FROM node:18 == instead of this you can use alpine/slim images becoz if you use node:18 its size will be in GB buth alpine will be in MB 
WORKDIR /app ==  application directory, if this does not exist docker will create this
COPY .. == Copy everything in current context, like whatever project files we have make sure to create image
RUN npm install == going to install all dependencies and store in new folder node_modules this folder will be very large in size as it contains all dependencies
RUN npm run build == going to create new folder like **app or dist**, it contains actual package that we will use to run the application
CMD ["node", "dist/app.js"] == this will start the application app.js will be entry point of application.

The main folder we need is dist or app whichever gets created, we don't need to have node_modules folder, for singlestage the if we keep node-modules foldeeeeer the size of image will be very big, so we need to have system in whihc we are able to get or use only app.js directory. To do that we will crate multistage docker file in which we can execute these(Single stage) cmnds, so that we can have dist folder created along with other things that we don't need like node_modules, once we have run this as stage1, then we create another cmnds as stage2, from stage1 we will copy dist/app.js  because we already ran npm install and build cmnds. so we can individually copy dist/app.js in stage2 and we can use it. so here benefit we are getting is in final docker stage2 image we will only have files that we actually need.
How we can implement this? 
# === STAGE 1: Build ===
FROM node:18 AS builder == Defining as builder becozz later when we want to copy from stage we can define i want to copy from builder stage
WORKDIR /app              # Sets working directory inside container
COPY package*.json ./ == we are copying this becoz we could utilize caching also
RUN npm install == when we run this it will check above copied file and based on that it will download all dependencies. when we want to run same for another image we o need to run these again we can cache these 
                   layers and call these cache in another run
COPY . .                  # Copies all source code to /app
RUN npm run build         # Builds the app (creates /app/dist or similar) , Till stage1 we are downloading all dependencies npm install copying everything and running npm run build, after this it will create dist folder along with other folders like src node_modules etc.but we need only dist folder
# === STAGE 2: Production ===
FROM node:18-slim == Smaller Node.js image for lightweight runtime == taking base image
WORKDIR /app == defining directory
# Copy only build artifacts and minimal files from builder stage
COPY --from=builder /app/dist ./dist == copy from build stage
COPY --from=builder /app/package*.json ./
RUN npm install --only=production == Installs only production dependencies
CMD ["node", "dist/app.js"]         # Starts the built app

Multistage Docker file client/frontend
======
# Stage 1: Build React App
FROM node:22-alpine AS builder == alpine based images will be small in size so using alpine as builder means stage is builder
WORKDIR /app == directory
# Copy dependency files first to leverage Docker cache
COPY package.json package-lock.json ./ == copying dependencies pkg.json will contain dependencies we will be needing pkg-lock.json will contains specific version that will be using
# Install dependencies (CI style = clean install from lockfile)
RUN npm ci == npm ci is better than install cmnd becoz it will focus on pkglock.json from above step so that ut can download exact version of dependency
# Copy the rest of the source code
COPY . .
# Build the React app (outputs to /build)
RUN npm run build == will generate static file of frontend that we will deploy it can be npm run dist/app/build
# Stage 2: Serve with Nginx
FROM nginx:alpine
# Copy built static files from builder stage to nginx html directory
COPY --from=builder /app/build /usr/share/nginx/html == copying static files from npm build cmnd copy at this location inside nginix /usr/share/nginx/html
# Optional: add custom nginx config (useful for SPA routing)
COPY nginx/default.conf /etc/nginx/conf.d/default.conf == final thing to copy is config file which we have to creatre in our root dorectory
# Expose port 80
EXPOSE 80 == provide info where to expose
# Start Nginx
CMD ["nginx", "-g", "daemon off;"] == start nginix deamon=off cmnd says run nginix in frontend daemon is somrthing running in background , stage2 is frontend docker image

we need 2 stages becoz in stage 1 we will be downloading all dependencies, biulding application and generating specific pkg taht we are using for deployment, out of these many things the only thing we actually need is actual package, But if you are building single stage docker file all these things will exist in that image which is going to make image very huge in size. Instaed of this we can have two stages, first stage will be builder stage where we are going to build application download dependencies create packages that will be needed, second stage will be actual stage where the application will be running, here what we can do from stage1 we can copy only one thing which is actual package that we want to deploy or use and rest we can writex: if we want to use ngnix we can write in stage2. so benefit you get final docker image will be only stage2 components and less in size, first thing that will create when you install dependencies is node_mmodules which is very huge in size and if you keeping it the image will become very huge so we will ignore thius and in second stage we copy only required ones, this will make sure that unnecesary things wont exist in final docker image, this is the main reason to use multistage docker files









 





 
