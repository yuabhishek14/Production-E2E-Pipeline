# Complete Production End to End Pipeline

This is a complete end to end declarative pipeline 

## Tech Stack

<p align="center">
  <img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" alt="GitHub Logo" width="70" height="70">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="60" height="70">
  <img src="https://maven.apache.org/images/maven-logo-black-on-white.png" alt="Maven Logo" width="180" height="50">
  <img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/2a79888f-1aa6-4d88-94c3-0b34bbde4099" alt="SonarQube Logo" width="230" height="80">
  <img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/80447647-d723-42fc-8b60-209d9a511115" alt="Docker Logo" width="100" height="80">
  <img src="https://roadie.io/static/59fa7a64fb19816e69c0c3dbd5d74a6c/29111/argo-cd-logo.webp" alt="Argo CD Logo" width="90" height="90">
  <img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" alt="Kubernetes Logo" width="250" height="50">
</p>

- **GitHub**: Version control system for source code management. 
- **Jenkins**: Automation server for building, testing, and deploying software.
- **Maven**: Build automation and project management tool for Java projects.
- **SonarQube**: Continuous inspection of code quality and security.
- **Docker**: Containerization platform for building, shipping, and running applications in containers.
- **Argo CD**: Continuous Delivery tool for Kubernetes applications.
- **Kubernetes**: Open-source container orchestration platform for automating application deployment, scaling, and management.


## Requirements

Before setting up the project, ensure you meet the following prerequisites:

1. **VirtualBox:**
   - Install VirtualBox on your system by downloading it from the official VirtualBox website: [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads).
   - Follow the installation instructions for your operating system to complete the setup.

2. **Virtual Machines:**
   - Create five virtual machines with the specified hostnames and purposes as follows:

   | Virtual Machine  | Hostname       | Purpose                                    | Private IP       |
   |------------------|----------------|--------------------------------------------|-----------------|
   | VM1              | jenkinsMachine | Jenkins server for CI/CD pipeline         | 192.168.1.2     |
   | VM2              | agentMachine   | Jenkins agent for building and executing jobs | 192.168.1.3               |
   | VM3              | sonar          | SonarQube server for code quality analysis | 192.168.1.4     |
   | VM4              | argoCd         | Argo CD server for Kubernetes continuous delivery | 192.168.1.5              |
   | VM5              | app-cluster    | Kubernetes cluster to deploy and manage applications | 192.168.1.6  |

   - To create the virtual machines, launch VirtualBox and follow these general steps:
     - Click on "New" to create a new virtual machine.
     - Provide a name and select "Linux" as the operating system, and choose "Ubuntu (64-bit)" as the version for each virtual machine.
     - Allocate sufficient memory (RAM) and create a virtual hard disk for each machine.
     - Configure the network settings with a "Bridged Adapter" to enable direct communication between the VMs and your host system.
     - Start each virtual machine and install Ubuntu 22.04 as the operating system on them.

3. **SSH Setup:**
   - Set up SSH access between your host system and the virtual machines to enable seamless communication.

## Jenkins Setup
Login to your VM1 and apply the following commands : 
#### Update Package Repository and Upgrade Packages
  Become root
```bash
sudo su -
```
  
  Run from shell prompt

```bash
sudo apt update
sudo apt upgrade
```
#### Adoptium Java 17
  Add Adoptium repository
```bash
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

```
  
  Update repository and install Java 17

```bash
apt update
apt install temurin-17-jdk
/usr/bin/java --version
exit 

```
#### Install Jenkins
  First, add the repository key to the system:
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
#### Starting Jenkins
  Let’s start Jenkins by using systemctl:
```bash
sudo systemctl start jenkins
```
  If everything went well, the beginning of the status output shows that the service is active and configured to start at boot:
```bash
Output
● jenkins.service - LSB: Start Jenkins at boot time
   Loaded: loaded (/etc/init.d/jenkins; generated)
   Active: active (exited) since Fri 2020-06-05 21:21:46 UTC; 45s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 1137)
   CGroup: /system.slice/jenkins.service
```
#### Access Jenkins User Interface

To set up your installation, visit Jenkins on its default port, 8080, using your server domain name or IP address: http://192.168.1.2:8080


## Jenkins SSH Based Agent Setup
Login to your VM2 and apply the following commands : 

#### Update Package Repository and Upgrade Packages
```bash
sudo su -
sudo apt update
sudo apt upgrade
```
#### Create Jenkins User and set Password
```bash
sudo adduser jenkins
```
Grant Sudo Rights to Jenkins User
```bash
sudo usermod -aG sudo jenkins
```
#### Adoptium Java 17
  Add Adoptium repository
```bash
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

```
  
  Update repository and install Java 17

```bash
apt update
apt install temurin-11-jdk
update-alternatives --config java
/usr/bin/java --version
exit 
```
#### Docker Setup
Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
Add Docker’s official GPG key:

```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
Use the following command to set up the repository:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Manage Docker as a non-root user
  Create the docker group
```bash
sudo groupadd docker
```
  Add your user jenkins to the docker group.
```bash
sudo usermod -aG docker jenkins
```
Change user to jenkins
```bash
su - jenkins
```
Run the following command to activate the changes to groups:
```bash
newgrp docker
```
Verify that you can run docker commands without sudo.
```bash
docker run hello-world
```

#### Connect to Remote SSH Agent
  From the Jenkins UI to Agent Machine IP
```bash
ssh jenkins@192.168.1.3
```
Create private and public SSH keys. The following command creates the private key jenkinsAgent_rsa and the public key jenkinsAgent_rsa.pub. It is recommended to store your keys under ~/.ssh/ so we move to that directory before creating the key pair.
```bash
mkdir ~/.ssh; cd ~/.ssh/ && ssh-keygen -t rsa -m PEM -C "Jenkins agent key" -f "jenkinsAgent_rsa"
```
Go to VM1 terminal and create .ssh/known_hosts file .
```bash
cd /var/lib/jenkins/
ssh jenkins@192.168.1.3
```
Switch back to VM2 terminal and Add the public SSH key to the list of authorized keys on the agent machine
```bash
cat jenkinsAgent_rsa.pub >> ~/.ssh/authorized_keys
```
Ensure that the permissions of the ~/.ssh directory is secure, as most ssh daemons will refuse to use keys that have file permissions that are considered insecure:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys ~/.ssh/jenkinsAgent_rsa
```
Copy the private SSH key (~/.ssh/jenkinsAgent_rsa) from the agent machine to your OS clipboard
```bash
cat ~/.ssh/jenkinsAgent_rsa
```
Now you can add the Agent on the Jenkins UI (Controller)

#### Agent Setup from Jenkins UI
Go to credentials :-

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/fbeb34c4-5864-41a1-b1ff-9f1939496505" alt="image" width="800" height="500" />

Go to the below path and change the Number of executors to 0 :-

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/df86fd99-eea1-41ea-934b-bb92136ca023" alt="image" width="500" height="500" />

Now Create a new Agent (remember to select permanent agent flag)
Fill the details as follows :-

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/01328e71-2823-4ab1-99c2-aa673466e266" alt="image" width="300" height="500" />

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/423f36b4-fca2-4257-ad24-a16898d93429" alt="image" width="300" height="500" />

## Jenkins Job (Basic)
#### Install Java and Maven tool on UI
Go to plugins and install these plugins : 

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/f72c392f-52d3-4602-a1bd-ef408cb05ed5" alt="image" width="300" height="250" />

#### Configure Tools on UI
1. Go to Tools and search for Maven
Note :- The Name field value should match with the Jenkinsfile maven tool value

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/c609470f-3b7e-4b0e-828a-29ef4ac0f5c4" alt="image" width="300" height="550" />

2. Now search for JDK

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/b9e7ee5d-1e60-4278-a23a-655c559b44fa" alt="image" width="300" height="550" /> 

3. Setup the Github credentials
   Note :- Generate personal access token and use it as password.

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/a51b27a7-0e73-47ed-9770-1334f23de9dd" alt="image" width="300" height="550" />

#### Jenkinsfile
Prepare the Jenkinsfile as follows :

```bash
pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'java17'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspaces"){
             steps{
                cleanWs()
             }
        }

        stage("Checkout from SCM"){
             steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/yuabhishek14/Production-E2E-Pipeline'
             }
        }

        stage("Build Application"){
             steps{
                sh "mvn clean package"
             }
        }

        stage("Test Application"){
             steps{
                sh "mvn test"
             }
        }
    }    
}
```
#### Pipeline from UI
Create a declarative jenkins job as follows :
1. Check the "Discard old builds" and "Max # of builds to keep" as 2

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/9ee4a4eb-e85e-435c-bb53-996e66c08b9c" alt="image" width="350" height="550" />

and run the pipeline , after successful run it will look something like this :

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/217be7c4-8ace-4166-9a85-14788d25c06c" alt="image" width="730" height="230" />

## SonarQube Integration

Login to your VM3 and execute the following commands : 
#### Install Postgresql 15
```bash
sudo apt update
sudo apt upgrade

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null

sudo apt update
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql
```

#### Create Database for Sonarqube

```bash
sudo passwd postgres
su - postgres

createuser sonar
psql 
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
\q

exit
```

#### Install Java 17

```bash
sudo su -

apt install -y wget apt-transport-https
mkdir -p /etc/apt/keyrings

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc

echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

apt update
apt install temurin-17-jdk
update-alternatives --config java
/usr/bin/java --version

exit 
```
#### Increase Limits

```bash
sudo vim /etc/security/limits.conf
```

Paste the below values at the bottom of the file

```bash
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

Now go to sysctl.conf file

```bash
sudo vim /etc/sysctl.conf
```

Paste the below values at the bottom of the file

```bash
vm.max_map_count = 262144
```

Reboot to set the new limits

```bash
sudo reboot
```

#### Install Sonarqube

```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt install unzip
sudo unzip sonarqube-9.9.0.65466.zip -d /opt
sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```

Update Sonarqube properties with DB credential

```bash
sudo vim /opt/sonarqube/conf/sonar.properties
```

Find and replace the below values, you might need to add the sonar.jdbc.url

```bash
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

Create service for Sonarqube

```bash
sudo vim /etc/systemd/system/sonar.service
```

Paste the below into the file

```bash
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

Start Sonarqube and Enable service

```bash
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
sudo tail -f /opt/sonarqube/logs/sonar.log
```

Access the Sonarqube UI
```bash
http://<IP>:9000 i.e http://192.168.1.4:9000
```

#### Generating Token

Go to My Account >> Security

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/49f576ab-3948-428c-888c-42fd18097b39" alt="image" width="740" height="260" />

#### Sonarqube credential
Add credentials on Jenkins UI and use the Token generated above as 'secret'

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/ce6d5493-024d-42e9-b7a0-396e75a99fee" alt="image" width="390" height="360" />

#### Install Required Plugins for SonarQube

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/c9aa8144-2950-4664-a894-c7bb66d63cc7" alt="image" width="390" height="380" />

#### Configure the Integration
Go to Configure system and in the sonarqube servers section do the following : 
Image 1

Now go to Global Configuration tool

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/4521fcb9-2e1b-4cd5-a96b-31f2d46d8c6d" alt="image" width="300" height="500" /> 

#### SonarQube Stage in Pipeline
Add the stage and run the pipeline
Updated script :
```bash
pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'java17'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspaces"){
             steps{
                cleanWs()
             }
        }

        stage("Checkout from SCM"){
             steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/yuabhishek14/Production-E2E-Pipeline'
             }
        }

        stage("Build Application"){
             steps{
                sh "mvn clean package"
             }
        }

        stage("Test Application"){
             steps{
                sh "mvn test"
             }
        }

        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }
    }    
}
```
once the pipeline run is complete you will get the result like this on Sonarqube

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/90b8f652-6554-466a-8617-a3a5bf0b1a56" alt="image" width="800" height="100" /> 

#### Quality Gate Stage in Pipeline
This stage is for if the project doest pass the quality it will block the deployment . For this we need a webhook to respond back to jenkins
Go to Adminstration >> Configuration >> Webhooks and click on create and fill the following :

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/3d5b8b8e-5216-44a8-94af-5a6a14026449" alt="image" width="360" height="400" /> 
