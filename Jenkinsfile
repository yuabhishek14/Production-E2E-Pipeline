pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "Production-E2E-Pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = ""
        DOCKER_PASS = ""
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN") 
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

        stage("Build & Push Docker Image"){
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
    }
}
