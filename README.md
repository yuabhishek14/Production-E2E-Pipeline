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

Now add the stage in script , updated script :
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

        stage("Sonarqube Analysis") {
            steps {
                script {
                    waitForQualityGate abortPipeline : false , credentialsId: 'jenkins-sonarqube-token'
                }
            }

        }
    }    
}
```
## Docker Integration
#### Install Plugins

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/ee741378-54b8-4601-8d90-0da02cfc0192" alt="image" width="360" height="400" /> 

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/15a8d715-008a-4118-a32d-d34bba02ee4e" alt="image" width="600" height="200" /> 

#### Add Docker Credentials 

Generate a Dockehub Access Token which will be used as a "Password"

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/8b24330a-4b0d-4ad7-87e7-6138a6b41d68" alt="image" width="150" height="400" />

#### Add the "Build & Push Docker Image"

Introduce Environment variable in the pipeline :

```bash
environment {
        APP_NAME = "production-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "abhishekdevops14"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
```

Create a Dockerfile 

```
FROM maven:3.9.0-eclipse-temurin-17 as build
WORKDIR /app
COPY . .
RUN mvn clean install

FROM eclipse-temurin:17.0.6_10-jdk
WORKDIR /app
COPY --from=build /app/target/demoapp.jar /app/
EXPOSE 8080
CMD ["java", "-jar","demoapp.jar"]
```

Add new stage in pipeline :

```bash
stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
```

## Kubernetes Integration 
Login to your VM4 and VM5 and execute the following commands parallelly: 

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

#### Create Kubernetes Cluster

```bash
sudo bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --disable traefik
exit
```

This command downloads and installs K3s, the lightweight Kubernetes distribution.

- `INSTALL_K3S_EXEC="server"`: Specifies that this instance will be the Kubernetes server.
- `--disable traefik`: Disables the built-in Traefik ingress controller, which is commonly used for managing incoming network traffic to applications.

Create Kubernetes Configuration Directory

```bash
mkdir .kube
```
- This command creates a directory named .kube in the user's home directory to store Kubernetes configuration files

Copy K3s Configuration
 ```bash
sudo cp /etc/rancher/k3s/k3s.yaml ./config
```

This command copies the K3s configuration file k3s.yaml from the default location /etc/rancher/k3s to the newly created .kube directory.

Change Ownership and Permissions
```bash
sudo chown ubuntu:ubuntu config
```

- This is done to ensure that the user has the necessary permissions to access and modify the configuration file.

Set Configuration File Permissions:

```bash
chmod 400 config
```

- This command sets the permissions of the configuration file to read-only for the owner. This enhances security by preventing unintended modifications.

Set KUBECONFIG Environment Variable

```bash
export KUBECONFIG=~/.kube/config
```

- This command sets the KUBECONFIG environment variable to point to the location of the K3s configuration file. This ensures that the kubectl command knows where to find the configuration for interacting with the Kubernetes cluster.

#### Install ArgoCD

```bash
kubectl create namespace argocd
```

- create namespace argocd: This command instructs Kubernetes to create a new namespace named "argocd." In Kubernetes, namespaces are used to isolate resources and workloads. Creating a dedicated namespace for ArgoCD helps keep its resources organized and separate from other applications or services.

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
  
- `apply -n argocd -f`: This command instructs Kubernetes to apply a configuration from a file.

- `-n argocd`: The `-n` flag is used to specify the namespace in which to create or apply the resources. In this context, it specifies the "argocd" namespace, ensuring that the resources are created within that namespace.

- `-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`: This part of the command specifies the URL of a YAML file containing the configuration manifest for ArgoCD. Kubernetes utilizes this file to generate the necessary resources for ArgoCD, such as deployments, services, and pods. The manifest file is hosted on GitHub within the ArgoCD project's repository.

#### Change Service to NodePort

To modify the service type from ClusterIP to NodePort, follow these steps:

    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
    ```

This command will update the service type for the ArgoCD server from ClusterIP to NodePort in the "argocd" namespace.

- `patch svc argocd-server`: Specifies that you want to patch a service named `argocd-server`.
- `-n argocd`: Specifies the namespace `argocd` where the service is located.
- `-p`: Indicates that you are providing a JSON or YAML patch for the resource.
- `'{"spec": {"type": "NodePort"}}'`: The JSON patch specifying that you want to change the service's type to NodePort.
- 
#### Get ArgoCd port
Now to access the ArgoCD UI we can check the port no by :
```bash
kubectl -n argocd get service 
```

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/0d1bb74f-25f5-440e-9e3c-574f8ef3bf77" alt="image" width="600" height="130" />

or you can check the port directly by doing :
```bash
kubectl -n argocd get service argocd-server -o jsonpath='{.spec.ports[0].nodePort}'
```

#### Fetch Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- **-n argocd:** The `-n` flag specifies the namespace where you want to perform the operation. In this case, it's the "argocd" namespace.

- **get secret argocd-initial-admin-secret:** This part of the command is used to retrieve a secret named "argocd-initial-admin-secret" in the specified namespace. Secrets in Kubernetes are used to store sensitive information securely.

- **-o jsonpath="{.data.password}":** The `-o` flag specifies the output format, and in this case, it's using JSONPath to format the output. JSONPath is a query language for JSON. This part is instructing Kubernetes to extract the value of the "password" field from the secret's data.

- **|:** This is a pipe operator, and it's used to pass the output of the previous command as input to the next command.

- **base64 -d:** This part of the command is responsible for decoding a base64-encoded string. The output of the previous command is a base64-encoded password, and this step decodes it to reveal the actual password.

Retrieve the password from the previous command and note the ArgoCD NodePort. Then, access the Argo UI by visiting http://VM5_IP:ArgoCD_Port in your web browser

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/8de5716d-b4c9-43d2-b1d8-aa68239c6062" alt="image" width="550" height="350" />

####

