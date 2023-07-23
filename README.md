# Complete Production End to End Pipeline

This is a complete end to end declarative pipeline 


## Tech Stack

<p align="center">
  <img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" alt="GitHub Logo" width="60" height="60">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="50" height="60">
  <img src="https://maven.apache.org/images/maven-logo-black-on-white.png" alt="Maven Logo" width="150" height="50">
  <img src="https://cdn.worldvectorlogo.com/logos/sonarqube.svg" alt="SonarQube Logo" width="100" height="60">
  <img src="https://1000logos.net/wp-content/uploads/2021/11/Docker-Logo-2013.png" alt="Docker Logo" width="100" height="50">
  <img src="https://roadie.io/static/59fa7a64fb19816e69c0c3dbd5d74a6c/29111/argo-cd-logo.webp" alt="Argo CD Logo" width="70" height="70">
  <img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" alt="Kubernetes Logo" width="200" height="50">
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

      
