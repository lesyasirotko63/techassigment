# Kubernetes Quickstart with Docker

This tutorial will guide you through the steps needed to package a simple Python web application into a Docker image and deploy it to a Kubernetes cluster. 

## Environment Setup

Before you start working with this project, it is essential to have the following tools installed and properly configured:

- [Docker](https://docs.docker.com/get-docker/)
- [Kubernetes (kubectl)](https://kubernetes.io/docs/tasks/tools/)
- [Minikube (for local Kubernetes deployment)](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download)
- [Git](https://git-scm.com/downloads)

## Step 1: Set Up the Project Directory

Create following directory and subfolders in Docker:

```bash
mkdir -p quickstart_docker/application
mkdir -p quickstart_docker/docker/application
```

Directory structure example:

```bash
    quickstart_docker/
    ├── application/     # Application code goes here
    └── docker/          # Docker-related files go here
        └── application/ # Dockerfile for the application
```

## Step 2: Create a Simple Application

Inside the `quickstart_docker/application` directory, create a Python web server, which will be example of our simple application. 

Create a file named `application.py` and add the following code:

```python
# application.py
import http.server
import socketserver

PORT = 8000
Handler = http.server.SimpleHTTPRequestHandler

httpd = socketserver.TCPServer(("", PORT), Handler)
print("Serving at port", PORT)
httpd.serve_forever()

```

The application requires a runtime environment, which means Python is needed, along with an operating system and specific dependencies. To save time, a pre-built image from Docker Hub will be used instead of building everything from scratch.

## Step 3: Create a Dockerfile

Next, package the application into a Docker image. To do this, create a Dockerfile inside `quickstart_docker/docker/application`:

```bash
touch quickstart_docker/docker/application/Dockerfile
```

Open the `Dockerfile` and add the following content:

```docker
# Use base image from the registry
FROM python:3.5

# Set the working directory to /app
WORKDIR /app

# Copy the 'application' directory contents into the container at /app
COPY ./application /app

# Make port 8000 available to the world outside this container
EXPOSE 8000

# Execute 'python /app/application.py' when container launches
CMD ["python", "/app/application.py"]
```

## Step 4: Build the Docker Image

When application and Dockerfile ready, it's time to build the Docker image. Run the following command from the `quickstart_docker` directory:

```bash
docker build . -f docker/application/Dockerfile -t exampleapp
```

Arguments:

-   `.` specifies the build context (the current directory).
-   `-f docker/application/Dockerfile` points to the Dockerfile location.
-   `-t exampleapp` tags the image as `exampleapp`.

## Step 5: Verify the Docker Image

Once the build process is complete, Docker image needed to be verified:

```bash
docker images
```

Output example:

```bash
REPOSITORY             TAG             IMAGE ID            CREATED             SIZE
exampleapp             latest          83ioe0edc28a        2 seconds ago       154MB
python                 3.6             05stv8636w3f        6 weeks ago         154MB

```
More information about docker images you can find [here](https://docs.docker.com/engine/reference/builder/.).

## Step 6: Push Image to a Repository

After verifying that the image is created, push it to a container repository (like Docker Hub or your private repository) so it can be deployed in Kubernetes. 

Here’s how to push it to Docker Hub:

1.  Log in to Docker Hub:
    
    `docker login` 
    
2.  Tag the image for Docker Hub (replace `yourusername` with your Docker Hub username):

    `docker tag exampleapp yourusername/exampleapp` 
    
3.  Push the image to Docker Hub:
   
    `docker push yourusername/exampleapp` 
    
Image is ready to be deployed using Kubernetes.


## Setting Up Kubernetes

To run your Dockerized application on Kubernetes, you first need a Kubernetes cluster. You can use [Minikube](https://minikube.sigs.k8s.io/docs/start/) for local development, or deploy it on a cloud provider like Google Kubernetes Engine (GKE), AWS Elastic Kubernetes Service (EKS), or Azure Kubernetes Service (AKS).

### 1. **Start Minikube**

If you are using Minikube for local Kubernetes development, you can start it with the following command:

`minikube start` 

This will start a local Kubernetes cluster. Verify that it's running:

`kubectl cluster-info` 

You'll need `kubectl` (the Kubernetes command-line tool) installed for interacting with your cluster. If you haven’t installed it yet, follow the official [kubectl installation guide](https://kubernetes.io/docs/tasks/tools/).

----------

## Creating Kubernetes Manifests

Kubernetes uses YAML files to describe the desired state of your application, including deployments, services, and pods. These are called Kubernetes manifests.

### 1. **Create a Deployment Manifest**

A **Deployment** ensures that a specified number of replicas of your application are running in the cluster. Create a file named `deployment.yaml` in your project folder and add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exampleapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: exampleapp
  template:
    metadata:
      labels:
        app: exampleapp
    spec:
      containers:
      - name: exampleapp
        image: yourusername/exampleapp:latest
        ports:
        - containerPort: 8000
``` 

For more information on Kubernetes deployments, check the official [Deployment documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### 2. **Create a Service Manifest**

A **Service** is used to expose your application to the network, allowing external access to it. Create a file named `service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: exampleapp-service
spec:
  selector:
    app: exampleapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer
```

For more details on Kubernetes services, see the official [Service documentation](https://kubernetes.io/docs/concepts/services-networking/service/).

----------

## Deploying an Application in Kubernetes

Once you've created the Kubernetes manifests, you can deploy your application using `kubectl`.

### 1. **Apply the Deployment and Service**

Run the following commands to apply the manifests and deploy your application:

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
``` 

This will create the deployment and service in your Kubernetes cluster. To check the status of your deployment:

`kubectl get deployments` 

To see the running pods:

`kubectl get pods` 

For local testing with Minikube, run this command to access the service:

`minikube service exampleapp-service` 

This command will open the application in your default web browser using the Minikube IP and service port.

### 2. **Scaling the Application**

You can scale your application by changing the number of replicas in your `deployment.yaml` file or by running the following command:

`kubectl scale deployment exampleapp-deployment --replicas=5` 

This will scale your application to 5 pods. Check the updated pod status:

`kubectl get pods` 

### 3. **Monitor the Application**

To check the logs from your pods:

`kubectl logs <pod-name>` 

To get more details on your deployment or troubleshoot:

`kubectl describe deployment exampleapp-deployment` 

For more information on managing applications in Kubernetes, check the [Kubernetes documentation](https://kubernetes.io/docs/home/).