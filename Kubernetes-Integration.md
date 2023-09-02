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

####


