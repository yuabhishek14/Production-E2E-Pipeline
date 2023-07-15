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
    }

    stages{
        stage("Checkout from SCM"){
             steps{
                git bransch: 'main', credentialsID: 'github', url: 'https://github.com/yuabhishek14/Production-E2E-Pipeline'
             }
        }
    }
}