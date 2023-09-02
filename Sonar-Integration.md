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