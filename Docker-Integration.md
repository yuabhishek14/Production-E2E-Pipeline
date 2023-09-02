## Docker Integration
#### Install Plugins

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/ee741378-54b8-4601-8d90-0da02cfc0192" alt="image" width="360" height="400" /> 

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/15a8d715-008a-4118-a32d-d34bba02ee4e" alt="image" width="600" height="200" /> 

#### Add Docker Credentials 

Generate a Dockehub Access Token which will be used as a "Password"

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/8b24330a-4b0d-4ad7-87e7-6138a6b41d68" alt="image" width="200" height="400" />

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
