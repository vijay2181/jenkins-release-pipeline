# jenkins-release-pipeline

we will get all branches from git repo and build the pipeline based on selected branch

## Install jenkins
```
INSTALLING JENKINS:
===================
- take t2.medium amazon linux 2 instance
- in real time just tell them that we are using Memory Optimized jenkins instances -> r5a.xlarge or r5a.2xlarge

- Install java:
sudo dnf install java-17-amazon-corretto-devel -y      ---> includes jdk
sudo dnf install java-17-amazon-corretto -y            ----> includes only jre

- Donwload Jenkins
use LTS version
https://pkg.jenkins.io/redhat-stable/

- add jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install jenkins

- when we install jenkins, it will create a service called jenkins
systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins

- access jenkins
http://ip:8080


Install git
============
sudo yum install git

Install maven
=============
- we may have to install multiple maven verions, so on jenkins global tool configurations i will install maven
Dashboard > Manage Jenkins > Tools > Maven installations
Name: MAVEN3.9.5
Install automatically
Version: 3.9.5

Java17 project:
===============
- take java 17 project for maven build, because we have java17 installed for jenkins
https://github.com/vijay2181/java-maven-SampleWarApp.git
```


SONARQUBE:
==========

```
- sonarqube tool is for code scanning, quality checking, publish qulaity report to sonarweb
- we need to install sonarqube on a seperate server

Install sonarqube:
------------------
take t2.medium instance

sonarqube:
----------
sudo dnf install java-17-amazon-corretto-devel -y
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
sudo unzip sonarqube-10.4.1.88267.zip
- as a good security practice dont run sonarqube as root user, create a normal sonar user and provide him sudo privilages and run sonar with that user

sudo useradd sonar

visudo
sonar  ALL=(ALL)    NOPASSWD:ALL

sudo chown -R sonar:sonar sonarqube-10.4.1.88267
cd /opt/sonarqube-10.4.1.88267/bin/linux-x86-64

[ec2-user@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh start
Starting SonarQube...
Failed to start SonarQube.

- because you are running sonar with ec2-user, so switch to sonar user

sudo su - sonar
cd /opt/sonarqube-10.4.1.88267/bin/linux-x86-64

[sonar@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh start
Starting SonarQube...
Started SonarQube.
[sonar@ip-172-31-20-171 linux-x86-64]$ sh sonar.sh status
SonarQube is running (29383).

- enable 9000 port in SG 
- and access -> http://52.39.183.141:9000
- login -> username=admin, password=admin
- change admin password: admin123

if you get issues while bringing up sonar, check sonar.log es.log
if you run sonar as root and again start sonarqube with sonar user you will get issues
so Remove this folder and run again
rm -rf /opt/sonarqube-10.4.1.88267/temp

```

### login to snarqube and create project plus token for external(jenkins,cli) authentication into sonarqube:


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/a159248c-64a0-43ad-8c57-9bc7f722f6d6)



- create a new project(local project) for our code analysis report publish, so to this project we can publish our reports

  

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/b3a55c50-50a6-4e51-a1ea-a8db12ebc020)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/e5839ab2-742a-4fe2-8b74-81ddacda24c5)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/60eabeb2-2caa-473e-958d-79eb15f87bec)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/6da085f2-26b8-46c8-bc28-45163eb2ad8f)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/81a563d9-d99c-4031-b98f-6a18780d53ee)


- for this test_project we will publish static code anaylsis report
- next we will create a token for authentication

### sonarqube token

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/cfab7a44-2318-4db5-8fb6-02fea7d1b12c)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/1f969c55-0b4a-42ef-8942-1ad3b827337f)

- generate token and copy it


Sonar Scanner Options:
======================

```
SonarQube scanning can be performed using three approaches:

1. SonarQube Scanner CLI: This involves using the SonarQube Scanner command-line interface. Configuration details such as SonarQube server URL, login credentials, etc., are specified in a `sonar.properties` file.
   
2. Jenkins SonarQube Scanner Plugin: Integration with Jenkins can be achieved using the SonarQube Scanner plugin. This allows for seamless integration with Jenkins pipelines, automating the analysis process.
   
3. Maven SonarQube Plugin: For Maven projects, SonarQube analysis can be initiated using the `mvn sonar:sonar` command. Configuration details are typically specified in the `pom.xml` file within the `<properties>` section.

Example Configuration in `pom.xml`:
------------------------------------
<properties>
    <sonar.host.url>http://35.165.197.226:9000</sonar.host.url>
    <sonar.login>admin</sonar.login>
    <sonar.password>admin123</sonar.password>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>


using the 'mvn sonar:sonar' command is a valid approach, it's generally recommended to use the SonarQube Scanner for Maven plugin (sonar-maven-plugin) for more comprehensive integration with Maven. This plugin provides more control and flexibility over the analysis configuration within the Maven build lifecycle.

- While using the SonarQube Scanner CLI can indeed provide a generic approach that is not tied to a specific build tool, it may require additional setup and configuration outside of the project's build scripts. This can introduce complexities, especially in a CI/CD environment where you might want seamless integration with your existing build tool.
The choice between using the SonarQube Scanner CLI versus the specific integrations like the Maven plugin or Jenkins plugin depends on your project's needs, existing infrastructure, and preferences for tooling and automation.

- but to be generic like if build tool is maven, gradle,ant etc... so to support all, i will go with sonarqube scanner cli package

- we need to install this, so im going to install this on same jenkins server cli

```


install sonarcli scanner on jenkis server itself
-------------------------------------------------

```
login to jenkins master server

cd /opt/
sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
sudo unzip sonar-scanner-cli-5.0.1.3006-linux.zip

- now we need to inform sonarscanner where our sonarqube is running(server ip, port, token etc..), because sonarscanner can run independently, in sonar.properties file we need to mention

cd /opt/sonar-scanner-5.0.1.3006-linux/conf
sudo vi sonar.properties
-------------------------------------------------------------------------------------------------------
#----- Default SonarQube server
sonar.host.url=http://52.10.226.100:9000                	#this can be overriden at the command
sonar.token=sqa_1e72010ca912043557aa2bb7e37e3696f3333  		 #this can be overriden at the command
sonar.projectKey=test_project
sonar.projectname=test_project
sonar.sources=src/main/java          				#src location of project
sonar.java.binaries=target/classes   				#target location of project
--------------------------------------------------------------------------------------------------------

chmod +x /opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
sudo chown -R jenkins:jenkins sonar-scanner-5.0.1.3006-linux/

- we need to goto project directory(/var/lib/jenkins/workspace/ci-pr) and run sonar scanner from there where (src/main/java) (target/classes ) folders are present
- sonar scanner location -> /opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner
- so here we are using generic sonar scanner which can be used for maven,gradle,ant etc....

/opt, /tmp directory are more permissive, allowing Jenkins to execute the SonarScanner CLI without encountering permission denied errors
The jenkins user typically has limited permissions to execute binaries in the home directory of other users, such as /home/ec2-user,

```

Jenkins Pipeleine:
------------------

you can find the official docs for sample sonar stage for jenkins pipeline

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/4c18f9f9-cde5-4002-ac5a-8930ed174aa6)


```
https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/
```


### confifigure sonarqube token inside jenkins server

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/81321f89-26fe-4eaf-a6f7-3b019869080f)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/2a293279-3130-4407-a4e8-29bbec3809c4)



### dynamically list all git branches inside jenkins pipeline

```
we need to dynamically list all git branches -> for this purpose we need to install Active Choices plugin, in that use -> Active Choices Reactive Parameter
https://medium.com/@g4b1s/dynamically-list-git-branches-in-jenkins-job-parameter-3e6e849f8a98

Active Choices Reactive Parameter
name: branches
groovy script:

def gettags = ("git ls-remote -t -h https://github.com/vijay2181/java-maven-SampleWarApp.git").execute()
return gettags.text.readLines().collect { 
  it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\^\\{\\}", '')
}

```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/139a4720-b8b5-43d1-a0b5-fd08feeb1d78)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/d83c2998-fae1-4129-9423-30ca796e1c37)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/9fb6bf34-a855-4ed5-b649-e2acdb7ea8ed)

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/822edb4c-fce3-48c6-af02-b46e52622ce5)

- come down and apply
- we need to approve the script if necessary
- manage jenkins -> In process script approval -> approve the script
- go back to job and reload the page, you will get drop downs
- now execute below steps for git branches dynamic listing

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/90d3ba40-4027-4df3-a6bb-b23f82aa77ed)


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/8d962c74-6cfb-447d-ad93-b9e506a71c3e)





```
def gettags = ("git ls-remote -t -h https://github.com/vijay2181/java-maven-SampleWarApp.git").execute()
return gettags.text.readLines().collect { 
  it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\^\\{\\}", '')
}


pipeline {
    agent any

    stages {
        stage('print user selected branch'){
            steps{
                     echo "${params.branches}"
            }
        }
    }
}

```


![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/225c47d0-cb95-404f-b637-94115b072b16)

```
environment {
    PATH = "${tool 'MAVEN3.9.5'}"
}
```

![image](https://github.com/vijay2181/jenkins-ci-pr-pipeline/assets/66196388/45550432-4ccd-49bf-8bfb-c644438dc60b)





#### Final Release Pipeline


```
pipeline {
    agent any
    
    environment {
        MAVEN = "${tool 'MAVEN3.9.5'}/bin/mvn"
        SONAR_HOST_URL = 'http://54.186.90.185:9000'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_PROJECT_KEY = 'test_project'
        SONAR_PROJECT_NAME = 'test_project'
    }
    
    stages {

        stage('Print Selected Branch') {
            steps {
               echo "${params.branches}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${params.branches}", url: 'https://github.com/vijay2181/java-maven-SampleWarApp.git'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh '$MAVEN clean package'
            }
        }
        
        stage('Run SonarScanner CLI on Maven project') {
            steps {
                sh '/opt/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url="${SONAR_HOST_URL}" -Dsonar.token="${SONAR_TOKEN}" '
            }
        }
        
        stage('Sonar Status Check') {
            steps {
                sh '''
                    #!/bin/bash
                    echo "This is a shell script within a Jenkins pipeline stage"
                    response=$(curl -u "${SONAR_TOKEN}": "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}")
                    project_status=$(echo "$response" | jq -r '.projectStatus.status')
                    echo "Project Status: $project_status"
                    if [ "$project_status" == "OK" ]; then
                        echo "Project Status is OK."
                    else
                        echo "Project Status is ERROR."
                    fi
                '''
            }
        }
       stage('Deploy to Next Stages') {
          steps {
             echo "This code can be deployable"
          }
       }       
    }
}


```






