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

