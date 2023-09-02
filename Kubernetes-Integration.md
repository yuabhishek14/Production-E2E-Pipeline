## Kubernetes Integration 
Login to your VM4(argocd) and VM5 (app-cluster) and execute the following commands parallelly: 

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

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/0d1bb74f-25f5-440e-9e3c-574f8ef3bf77" alt="image" width="625" height="130" />

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

#### Add the App-Cluster in ArgoCd

Check the nodes on both VM4 and VM5, where the node names match the VM hostnames.

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/18566fe6-f6c8-4166-8e05-1db78204fc73" alt="image" width="860" height="130" />

Now we need to add the app-cluster to our argocd machine .

- Copy the config file content from the .kube folder on VM5.

- Create a file named "app-cluster.yaml" in the .kube folder on VM4 and paste the copied content into it.

- Change the IP address as this is the IP to which kubectl will talk and we copied it from the localhost of VM5
therefore we need to change the IP to VM5 IP and save the file

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/8a5d2f22-766d-4156-8a18-f12406f189ec" alt="image" width="960" height="270" />

```bash
export KUBECONFIG=~/.kube/app-cluster.yaml
```

Now if we check the node we are able to talk to app-cluster

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/77b77491-8677-4425-a53b-6ce2cc8c1b05" alt="image" width="450" height="130" />

#### Install ArgoCD command line tool

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

#### Login to ArgoCD

```bash
argocd login 192.168.1.5:31815 (NodeIP:argocd port)

```

#### Register App-Cluster

```bash
argocd cluster add default --name app-cluster
```

- After running these commands, you should be logged into the specified ArgoCD server, and the "app-cluster" should be added as a target cluster for deploying applications using ArgoCD. You can then use ArgoCD's GitOps features to define and manage your applications' desired states in Git repositories and have ArgoCD automatically synchronize and deploy those applications to the "app-cluster" Kubernetes cluster.

Now if we check the clusters section in ArgoCD will be able to see that our App-Cluster is added to ArgoCD

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/112e2385-1eec-4677-a58d-4dc5c2b9d19b" alt="image" width="1220" height="230" />

#### Create a Application

1. Create a separate repository where we will create "deployment" and "service" manifest

[GitOps Repo](https://github.com/yuabhishek14/gitops-production-e2e-pipeline)

2. Add the gitops repo in the ArgoCD (use the Github personal access token as password)

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/f36d9a23-5afe-4011-b431-d62d5d3013f5" alt="image" width="300" height="350" />

3. Click on create application and fill the information like this

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/a4cd1f31-30c5-4c3c-893e-462a6e5c6296" alt="image" width="260" height="380" />
<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/945c0ebe-6a9d-4b32-971b-c61e56245716" alt="image" width="280" height="380" />


- Now if we check the Application we will see 2 pods are created 

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/6872200c-9a7a-426c-b175-9ba2215f05aa" alt="image" />

#### Automate Image Versioning in Deployment manifest

Currently we have to manually update the image version in the deployment.yaml file in our gitops repo.
To automate this process 

Create Jenkins API Token

- Go to Jenkins UI and go to your user and click on configure
- Click on "Add new Token" . Name it as "JENKINS_API_TOKEN"

Add the JENKINS_API_TOKEN credentials

- Use the token generated earlier as the password.
<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/a119d51f-7b15-4d1f-9fc2-181eec6b068f" alt="image" width="280" height="380" />


Add new environment variable
```bash
environment {
JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
}
```
Add new stage in Jenkinsfile
```bash
stage("Trigger CD Pipeline") {
            steps {
                script {
                      sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.1.2:8080/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"

                    }
                }
            }
```

Create another pipeline named **"gitops-complete-pipeline"**

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/78f32cf6-907f-4d45-a2c6-c17311c590c4" alt="image" width="280" height="380"/>
<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/5390070c-2e6e-4703-8c22-72f7ac4c0639" alt="image" width="350" height="380"/>

<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/01c8efab-0b72-4ce1-83c5-b6faa5f0817e" alt="image" width="280" height="380"/>
<img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/560ef122-2a74-4896-b1e6-1e95babfb47c" alt="image" width="280" height="470"/>

Create the Jenkinsfile for gitops pipeline :

```bash
pipeline {
    agent {
        label "jenkins-agent"
    }
    environment {
        APP_NAME = "production-e2e-pipeline"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
    
    
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/yuabhishek14/gitops-production-e2e-pipeline'
            }
        }
    

    
        stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat deployment.yaml
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                    cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "yuabhishek14"
                    git config --global user.email "yuabhishek14@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/yuabhishek14/gitops-production-e2e-pipeline main"
                }
            }
        }

    }

}
```