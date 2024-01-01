# How to deploy to AWS EKS a SpringBoot WebAPI

## 1. Prerequisites

## 1.1. Install and run Docker Desktop

https://docs.docker.com/desktop/install/windows-install/

## 1.2. Install **eksctl**:

```
choco install eksctl
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/73e78a05-3ff8-4917-9be2-0a754145c199)

## 1.3. Run Docker 

Delete the config.json file in this location: C:\Users\luisc\.docker\config.json

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/f77100a5-dd66-45e1-80ce-03273a5b799c)



## 2. Create AWS ECR public repo




## 3. Create a Docker image and push it to Docker Desktop





To deploy your application in AWS Elastic Kubernetes Service (EKS), you'll need to create two YAML files: one for the deployment (**deployment.yml**) and one for the service (**service.yml**). 

Below are sample YAML files that you can use and modify according to your requirements.

**Deployment YAML (deployment.yml)**

This file defines the deployment of your application in the Kubernetes cluster.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demoapi-deployment
spec:
  replicas: 2  # The number of Pods to run
  selector:
    matchLabels:
      app: demoapi
  template:
    metadata:
      labels:
        app: demoapi
    spec:
      containers:
        - name: demoapi
          image: <your-docker-image>  # Replace with your Docker image, e.g., "username/demoapi:latest"
          ports:
            - containerPort: 8080
```

Replace **<your-docker-image>** with the path to your Docker image in your container registry.

**Service YAML (service.yml)**

This file defines how your application is exposed.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demoapi-service
spec:
  type: LoadBalancer  # Exposes the service externally using a load balancer
  selector:
    app: demoapi
  ports:
    - protocol: TCP
      port: 80  # The port the load balancer listens on
      targetPort: 8080  # The port the container accepts traffic on
```

**Deploying to AWS EKS**

After creating these files, you can use the kubectl command-line tool to apply these configurations to your EKS cluster:

Deploy the Application:

```
kubectl apply -f deployment.yml
```

**Create the Service**

```
kubectl apply -f service.yml
```

Verify the Deployment:

Check the status of the deployment:

```
kubectl get deployments
```

Check the status of the pods:

```
kubectl get pods
```

Access the Service:

Once the service is deployed, it will get an external IP address.

You can find the external IP address with:

```
kubectl get service demoapi-service
```

Use the external IP address to access your application in a web browser: **http://<EXTERNAL-IP>/hello**
