# Complete Production End to End Pipeline

This is a complete end to end declarative pipeline 

## Tech Stack

<p align="center">
  <img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" alt="GitHub Logo" width="70" height="70">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="60" height="70">
  <img src="https://maven.apache.org/images/maven-logo-black-on-white.png" alt="Maven Logo" width="180" height="50">
  <img src="data:image/png;base64,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" alt="SonarQube Logo" width="150" height="80">
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

## Jenkins Job
#### Intall Java and Maven tool on UI
GO to plugins and install these plugins : 

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/f72c392f-52d3-4602-a1bd-ef408cb05ed5" alt="image" width="300" height="250" />


