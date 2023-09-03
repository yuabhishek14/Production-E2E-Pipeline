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
   - Create five Ubuntu 22.04 virtual machines with the specified hostnames and purposes as follows:

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
4. **VDI Reuse**
   - You can directly download Ubuntu 22.04 VDI from Here [Link](https://www.linuxvmimages.com/images/ubuntu-2204/)
   - To reuse the same downloaded VDI again you have to update UUID since you cant have two VM with same UUID.
   - For this go to the virtualbox file location and open CMD at that location
   - Run this command (D:\VM\VM5\VM5.vdi is the location of the vdi whose uuid needs to be changed):

   ```bash
   .\VBoxManage.exe internalcommands sethduuid "D:\VM\VM5\VM5.vdi"
   ``` 
3. **SSH Setup:**
   - Set up SSH access between your host system and the virtual machines to enable seamless communication.


## Table of Contents

- [Jenkins Setup](#jenkins-setup)
- [Jenkins Agent Setup](#jenkins-agent-setup)
- [Sonar Integration](#sonar-integration)
- [Docker Integration](#docker-integration)
- [Kubernetes Integration](#kubernetes-integration)

---

## Jenkins Setup

[Detailed Documentation](https://github.com/yuabhishek14/Production-E2E-Pipeline/blob/main/Jenkins-Setup.md)

---

## Jenkins Agent Setup

[Detailed Documentation](https://github.com/yuabhishek14/Production-E2E-Pipeline/blob/main/Jenkins-AgentSetup.md)

---

## Sonar Integration

[Detailed Documentation](https://github.com/yuabhishek14/Production-E2E-Pipeline/blob/main/Sonar-Integration.md)

---

## Docker Integration

[Detailed Documentation](https://github.com/yuabhishek14/Production-E2E-Pipeline/blob/main/Docker-Integration.md)

---

## Kubernetes Integration

[Detailed Documentation](https://github.com/yuabhishek14/Production-E2E-Pipeline/blob/main/Kubernetes-Integration.md)
