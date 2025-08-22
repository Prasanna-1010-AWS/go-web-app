# End-to-End DevOps Implementation Guide for `go-web-app`

## Project Overview

This project demonstrates how to **“DevOpsify”** an existing application that currently has **no DevOps practices**.

- The development team has already created a project called `go-web-app` with:
  - Source code
  - Unit tests
  - Static content
  - Instructions on how to run the app

- The goal is to implement **full DevOps practices**, including:
  - CI/CD
  - Containerization
  - Kubernetes deployment
  - GitOps workflows

---

## DevOps Practices We Will Implement

1. **Containerization**
   - Create a **Dockerfile** using a **distroless image** for security and minimal footprint.

2. **Kubernetes Deployment**
   - Create **K8s manifests**:
     - `Deployment`
     - `Service`
     - `Ingress`
   - Use **EKS** (Elastic Kubernetes Service) as the cluster.

3. **Helm Charts**
   - Package the application into **Helm charts**.
   - Future-proof: instead of writing manifests for each environment, you can reuse the chart with **`values.yaml`** for different environments.

4. **Ingress Controller**
   - Configure **Ingress** to automatically create a **Load Balancer**.
   - Map the Load Balancer IP to a local DNS so the app is accessible externally.

5. **Continuous Integration (CI)**
   - Implement **GitHub Actions** with multiple stages:
     - Code checkout
     - Build
     - Test
     - Docker image creation

6. **Continuous Deployment (CD)**
   - Use **GitOps** principles with **ArgoCD** for automated deployments from Git.

---

## Outcome

By the end of this implementation:

- The `go-web-app` will be **fully containerized and deployed** to Kubernetes.
- CI/CD will automatically **build, test, and deploy** changes.
- Helm charts will allow **easy deployment to multiple environments**.
- Ingress will make the app **accessible via a domain or DNS**.
- This setup mirrors what a DevOps team would implement in a real organization.

## Understanding the Application  

Before implementing DevOps practices, a DevOps Engineer must understand the project in detail. This includes:  

- How the application should be **built**  
- How the application should be **run**  
- On which **port** the application is running  
- How the application **looks like** when accessed  

## Create a Project Folder  

Create a folder named **go-web-app** in your local machine using the following command:  

```bash
mkdir go-web-app

## 1. Clone the Repository  

Move into the **go-web-app** folder and clone the repository inside it:  

```bash
cd go-web-app
git clone <repository_url>

## 2. Build and Run the Application  

Build the Go application:  

```bash
go build -o main .

## 3. Verify the Binary File  

After building the application, list the files in the folder to verify the binary has been created:  

```bash
ls

## 4. Execute the Go Binary  

Run the compiled Go application using the binary:  

```bash
./main

## 5. Access the Application in Browser
Once the Go binary is running, open your web browser and navigate to:

http://localhost:8080/courses

- The application should now be visible and running.  
- According to the developer team's repo, the **server starts on port 8080**.  


## 6. Write the Dockerfile  

Open your project folder in VS Code to create the Dockerfile:  

```bash
code .

- This will open the go-web-app folder in VS Code.
- Inside VS Code, create a file named Dockerfile in the root of the project.

## 7. Create a Multi-Stage Dockerfile  

We will use a **multi-stage Dockerfile** to build the Go application and create a minimal, secure image using a distroless base.  

**Step 1: Build Stage**  
- Download all dependencies  
- Build the Go application  

**Step 2: Final Stage**  
- Use a **distroless image** as the base  
- Copy the compiled binary from stage 1  
- Expose the application port  
- Run the application  

## 8. Build the Docker Image 

Once the Dockerfile is created, build the Docker image using the following command: 

```bash
docker build -t prasanna/go-web-app:v1 .

## 9. Run the Docker Image  

Run the Docker container using the following command:  

```bash
docker run -p 8080:8080 -it prasanna/go-web-app:v1

## 10. DockerHub Login

First, log in to DockerHub from your local machine:  

```bash
docker login

## 11. Push the Docker Image to DockerHub  

After logging in, push your Docker image to DockerHub:  

```bash
docker push prasannadevops2025/go-web-app:v1

## 12. Create Kubernetes Manifests  

1. In VS Code, create a folder named **`k8s`** inside your `go-web-app` folder.  
2. Inside the `k8s` folder, create another folder named **`manifests`**.  
3. Inside the `manifests` folder, create the following files:  
   - `deployment.yaml`  
   - `service.yaml`  
   - `ingress.yaml`  

## 13. Prepare Kubernetes Cluster  

- To validate the Kubernetes manifests (`deployment.yaml`, `service.yaml`, `ingress.yaml`), you need a **Kubernetes cluster**.  
- We will create an **EKS (Elastic Kubernetes Service) cluster** on AWS for this purpose.  
- Once the cluster is ready, you can apply the manifests to deploy the application.  


## 14. Create EKS Cluster on AWS  

### Prerequisites (install in your local machine)  
- **kubectl** → Command-line tool to interact with Kubernetes clusters  
- **eksctl** → Simple CLI tool to create and manage EKS clusters  
- **AWS CLI** → Configure with your AWS credentials  
  ```bash
  aws configure
  # Provide Access Key, Secret Key, Region, Output format



## 15. Install an EKS Cluster with EKSCTL  

Run the below command to create an EKS cluster in AWS:  

```bash
eksctl create cluster --name go-web-app --region ap-south-1


## 16. Validate the Deployment YAML  

Apply the Deployment manifest to the EKS cluster:  

```bash
kubectl apply -f k8s/manifests/deployment.yaml

To confirm the pods are running:

```bash
kubectl get pods



## 17. Apply and Validate the Service  

Apply the Service manifest to the EKS cluster:  

```bash
kubectl apply -f k8s/manifests/service.yaml

Check the Service status:

```bash
kubectl get services

## 18. Apply and Validate the Ingress  

Apply the Ingress manifest to the EKS cluster:  

```bash
kubectl apply -f k8s/manifests/ingress.yaml

Check the Ingress status:

```bash
kubectl get ingress

## 19. Accessing Applications via Ingress  

- You **cannot directly access** applications by just creating an Ingress resource (`kubectl get ing`).  
- An **Ingress Controller** (e.g., NGINX Ingress Controller or AWS Load Balancer Controller) is required to process Ingress rules.  
- The Ingress Controller assigns an **external address (IP or hostname)** to the Ingress resource.  

### Steps:
1. Deploy an **Ingress Controller** in your EKS cluster.  
2. After deployment, check the Ingress resource:  
   ```bash
   kubectl get ingress


## 20. Verify the Service with NodePort  

Before proceeding further, verify that the Service is working correctly by exposing it as a **NodePort**:  

1. Edit the Service:  

```bash
kubectl edit svc go-web-app

- Change the type field to NodePort.

- Save and exit the editor.

Verify the Service is running on a NodePort:

```bash
kubectl get svc


## 21. Open Security Group Ports  

To allow external access to your NodePort service, update the EKS worker node **Security Group**:  

- Open the Security Group in the AWS console associated with your worker nodes.  
- Add a rule to allow **all traffic** (or at least TCP traffic on your NodePort range).  

This ensures that your NodePort service is reachable from outside the cluster.  


## 22. Access the Application Using NodePort  

After opening the Security Group ports:  

1. Get the NodePort assigned to your service:  

```bash
kubectl get svc

This shows the port number your service is exposed on.

2. Get the external IP address of a node:

```bash
kubectl get nodes -o wide

Access the application in your browser using:
http://<ExternalNodeIP>:<NodePort>

- Replace <ExternalNodeIP> with the node’s external IP.

- Replace <NodePort> with the port shown in kubectl get svc.

## 23. Configure Ingress Controller  

Install the **Nginx Ingress Controller** on AWS to manage external access to your services:  

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml

- This deploys the Nginx Ingress Controller in your cluster.

- The Ingress Controller will watch for Ingress resources and create a Load Balancer automatically to expose your applications externally.


## 24. How Ingress Controller Works  

- The **Ingress Controller** continuously watches the cluster for **Ingress resources**.  
- When it detects a new Ingress, it automatically:  
  - Creates a **Load Balancer**  
  - Routes external traffic to the appropriate service based on the rules defined in the Ingress YAML  
- This allows your application to be accessible via a domain name or external IP.  


## 25. Verify Ingress Controller  

Check the Ingress Controller pods and ensure it is running properly:  

```bash
kubectl get pods -n ingress-nginx

- This will display the pods of the Ingress Controller.

- Use the pod information to confirm that the Ingress resources are associated with the controller.

- Once associated, the Load Balancer will be provisioned and ready to route traffic to your application.

## 26. Verify Load Balancer Creation  

- After the Ingress Controller is running, check if the **Load Balancer** has been created.  
- Note: If you try to access the application using the Load Balancer link directly, it may **not work** because the **Ingress YAML is configured with a domain name**.  
- The Ingress routes traffic based on the hostname defined in the YAML.  

- To access the app properly, you need to map the Load Balancer IP to the domain name configured in the Ingress.  

## 27. Resolve Load Balancer IP with nslookup  

- Copy the domain name or URL from the Load Balancer.  
- Use `nslookup` to get the corresponding IP address:  

```bash
nslookup <your-domain-name>

- This command will return the IP address of the Load Balancer.

- Use this IP for testing or mapping to your local DNS.


## 28. Map Load Balancer IP to Local DNS  

- Open the `/etc/hosts` file with elevated privileges to map the Load Balancer IP to your domain:  

```bash
sudo vim /etc/hosts

write the IP and domain name as shown: <LoadBalancer-IP> <your-domain-name>

- Open your web browser and navigate to your **domain name** (as configured in Ingress YAML).  
- Your application should now be accessible and working correctly via the domain.  


## 29. Introduction to Helm  

### What is Helm?  
- **Helm** is a **package manager for Kubernetes**, similar to:  
  - `apt` for Ubuntu  
  - `npm` for Node.js  

### Why Helm is Used?  
- In Kubernetes, deploying applications often requires many resources:  
  - Pods, Deployments, Services, Ingress, Secrets, ConfigMaps, etc.  
- Writing all these YAML files manually and updating them for different environments (dev, staging, prod) is **repetitive and error-prone**.  
- **Helm** simplifies this process by allowing you to **install, upgrade, and manage complex applications** on Kubernetes using a single command.  

### Helm Chart  
- A **Helm Chart** is a package of Kubernetes YAML files (Deployments, Services, ConfigMaps, etc.) bundled together.  
- It uses **templates and configuration values** to make deployments reusable and environment-specific.  


## 30. Create Helm Chart for the Application  

1. Create a new folder for Helm charts:  

```bash
mkdir helm
cd helm

Create a Helm chart for go-web-app:

```bash
helm create go-web-app

This will generate a set of files and folders inside go-web-app.

Delete the charts folder, as it is not needed for now.

Important remaining files and folders:

Chart.yaml → Metadata about the chart (name, version, description)

templates/ → Contains Kubernetes resource templates (Deployment, Service, Ingress, etc.)

values.yaml → Default configuration values used by the templates

## 31. Clean Up Templates Folder  

- Navigate to the `templates` folder inside your Helm chart:  

```bash
cd helm/go-web-app/templates

Delete all default template files generated by Helm:

```bash
rm -rf *


## 32. Copy Kubernetes Manifests into Helm Templates  

- Copy your existing Kubernetes manifests into the Helm chart's `templates` folder:  

```bash
cp ../../../k8s/manifests/* .

This will move:

- deployment.yaml

- service.yaml

- ingress.yaml

Now the Helm chart uses your custom manifests as templates for deployment.

## 33. Install the Application Using Helm  

- Deploy your application using the Helm chart:  

```bash
helm install go-web-app ./go-web-app

- go-web-app → Name of the release

- ./go-web-app → Path to your Helm chart

- Helm will create all resources (Deployment, Service, Ingress) in your cluster based on the templates and values.yaml.

- You can now verify the deployment using:

kubectl get pods
kubectl get svc
kubectl get ingress

## 34. Uninstall Helm Release  

- If you need to remove the deployed application, use the `helm uninstall` command:  

```bash
helm uninstall go-web-app

- This will delete all resources (Deployment, Service, Ingress, etc.) created by the Helm chart for the go-web-app release.

## 35. Continuous Integration (CI) Using GitHub Actions  

Typical stages of a CI pipeline:  

1. **Build and Unit Test**  
   - Checkout code from GitHub  
   - Install dependencies  
   - Run unit tests  
   - Fail the pipeline if tests fail  

2. **Run Static Code Analysis**  
   - Tools: SonarQube, ESLint, Pylint, etc.  
   - Detect bugs, code smells, or style issues  

3. **Create Docker Image and Push to DockerHub**  
   - Build Docker image using `docker build`  
   - Tag the image (commit SHA or version)  
   - Push image to DockerHub (or any container registry)  

4. **Update Helm Chart with the Docker Image**  
   - Update `values.yaml` or use `--set image.tag=...`  
   - Commit and push updated `values.yaml` to the **GitOps repository** (the repo ArgoCD watches)  

This completes the CI workflow:
Code → Tested → Container → Updated Helm Chart


## 36. Continuous Deployment (CD) Using GitOps + ArgoCD  

- **ArgoCD** continuously monitors the Git repository containing Helm charts.  
- When `values.yaml` changes (e.g., a new Docker image tag), ArgoCD performs the following:  
  1. Detects the change  
  2. Pulls the Helm chart from the repository  
  3. Renders the templates using the updated values  
  4. Applies the rendered manifests to the Kubernetes cluster  

This completes the CD workflow:
Helm Chart Changes → ArgoCD Deploys → Kubernetes Cluster Updated


- **Note:** CI does **not deploy directly** to Kubernetes in this setup.  
- CI’s role is limited to: build, test, package, push Docker image, and update Helm chart.  

work continues