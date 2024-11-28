# Deploying Java Microservices on Kubernetes with Minikube: A Hands-On Guide

If you’re diving into the world of microservices and Kubernetes, you’ve come to the right place! Kubernetes is the go-to platform for orchestrating containers and microservices, and in this guide, I’m going to walk you through how to deploy microservices in a Kubernetes cluster using Minikube. Whether you're new to Kubernetes or looking to brush up your skills, this is a step-by-step journey from setting up your local Kubernetes cluster to accessing your microservices!

You will learn how to build Java applications with Maven, containerize them with Docker, push the Docker images to Docker Hub, and deploy the microservices to Kubernetes.

Ready? Let’s dive in.

## Prerequisites
Before we get started, there are a few things you’ll need:
- Java application source code
- Docker installed
- Kubernetes cluster (Minikube)
- Basic knowledge of Maven, Docker, and Kubernetes
 
If you already have these tools installed, fantastic! If not, don’t worry — I’ve got you covered!

## Microservices in the DJShopping Application

This example consists of the following microservices:

1. **Shopfront**: The primary entry point for end-users (Web UI and API-driven).
2. **Product Catalogue**: Provides product details like name and price.
3. **Stock Manager**: Provides stock information such as SKU and quantity.

---

## Steps to Deploy Java Microservices

### 1. Start Minikube
Minikube is a tool that lets you run a Kubernetes cluster locally on your machine, perfect for testing out microservices without needing a cloud provider. Let’s get our Minikube cluster up and running, and we’ll be good to go.

Start Minikube with the Docker driver:

```bash
minikube start --driver=docker
```
Once Minikube finishes setting up, you’ll want to check the status of your cluster. This is an important step to ensure everything is running smoothly.


To verify that your cluster is up and the Kubernetes control plane is working as expected, simply run: 

```bash
kubectl cluster-info
```

This will give you an overview of your Kubernetes cluster, showing you the services running under Minikube. If everything looks good, we’re ready for the next step.

### 2. Install Maven
Now, it’s time to get Maven installed. Maven is our build automation tool that will compile and package our Java-based microservices. Depending on your operating system, follow the instructions below:

#### For Linux-based Systems:
On Linux systems, installing Maven is super simple:

```bash
yum install maven
```

#### For Windows (with Chocolatey installed):
If you’re on Windows and have Chocolatey installed, you can get Maven with a single command:

```bash
choco install maven
```

After installation, set up Maven configuration to ensure the proper environment for building applications.

### 3. Build Java Application
With Maven installed, let’s build our microservices. Navigate to the directory containing your project’s pom.xml file and run:

```bash
mvn clean installed
```

This generates a target folder containing the .jar file for the application.

### 4. Create a Dockerfile
Next, we need to containerize the Java application. Docker helps us create containers that package the application and its dependencies, making it easy to deploy across various environments, including Kubernetes.

In the same directory as your pom.xml file, create a Dockerfile that defines how the application will be built and run inside a Docker container. Here's an example of what your Dockerfile might look like:
```bash
# Base image
FROM openjdk:8-jre
# Copy jar file into the container
ADD target/shopfront-0.0.1-SNAPSHOT.jar app.jar
# Expose port
EXPOSE 8010
# Set entry point
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
In this DockerFile:

- We start with an OpenJDK base image.
- We copy the shopfront jar file into the container.
- We expose port 9010 for the service to be accessible.
- The entry point is set to run the jar file inside the container using java.


### 5. Build and Push Docker Images

Once the Dockerfile is ready, build your Docker image by running:

```bash
docker build -t <dockerhub-username>/<microservice-name>:lastest .
```

Here, I'll show the example for shopfront: 

```bash
docker build -t mariamtahir/shopfront:latest .
```
You can verify the created docker images using:

```bash
docker images
```

Push the images to Docker Hub:
For Kubernetes to access your Docker image, we’ll need to push it to a container registry like Docker Hub.

To push your image, first log in to Docker Hub:
```bash
docker login
```

Then push your image to Docker Hub:
```bash
docker push <dockerhub-username>/<microservice-name>:lastest
```
Now your image is available online for Kubernetes to pull when deploying your microservices.

Repeat these steps for all microservices (shopfront, productcatalogue, stockmanager).

### 6. Deploy Microservices to Kubernetes
Now that your Docker images are ready and pushed to Docker Hub, let’s deploy them in Kubernetes.

#### Create a Kubernetes YAML File
Below is an example of shopfront-service.yaml:

```bash
---
apiVersion: v1
kind: Service
metadata:
  name: shopfront
  labels:
    app: shopfront
spec:
  type: NodePort
  selector:
    app: shopfront
  ports:
    - protocol: TCP
      port: 8010
      name: http

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopfront
spec:
  selector:
    matchLabels:
      app: shopfront
  replicas: 1
  template:
    metadata:
      labels:
        app: shopfront
    spec:
      containers:
        - name: shopfront
          image: mariamtahir/shopfront:latest
          ports:
            - containerPort: 8010
          livenessProbe:
            httpGet:
              path: /health
              port: 8010
            initialDelaySeconds: 30
            timeoutSeconds: 1
```

This file defines both a service and a deployment for the shopfront microservice. The service exposes the microservice within the Kubernetes cluster, and the deployment tells Kubernetes how to manage the container.

#### 7. Apply YAML Files
Apply the configuration file with:

```bash
kubectl apply -f shopfront-service.yaml
```

Do the same for the other microservices (like productcatalogue and stockmanager) using similar yaml files, adjusting the names and ports as needed

### 8. Verify Deployment
Check the pods, deployments, services:

```bash
kubectl get deployments
kubectl get svc
kubectl get pods
```

### 9. Access the Application
To access the application, retrieve the Minikubes service URL:

```bash
minikube service shopfront-service
```

This will open up your microservice in the browser. If everything has been set up correctly, you should see your service up and running!

## Conclusion
By following these steps, you can successfully deploy and access microservices on a Kubernetes cluster using Minikube. Each microservice runs in its container, and Kubernetes efficiently manages the pods, services, and scaling. This setup demonstrates the foundational concepts for deploying a microservices-based application on Kubernetes, giving you a solid understanding of the tools and techniques used in modern cloud-native architectures.











