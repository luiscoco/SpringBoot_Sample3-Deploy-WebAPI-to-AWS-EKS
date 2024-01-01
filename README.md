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

Run the command:

```
docker login
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/9149c4df-2a65-49da-905a-a4e036c25960)

Delete the letter "s" in the word "credsStore" 

```
{
	"auths": {
		"https://index.docker.io/v1/": {}
	},
	"credsStore": "wincred"
}
```

This is the new config.json file

```
{
	"auths": {
		"https://index.docker.io/v1/": {}
	},
	"credStore": "wincred"
}
```

## 2. Create AWS ECR public repo

We navigate to AWS ECR and we create a new public repo

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/92d7a450-69f0-47ed-abde-9035fc19a41e)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/673202ed-1071-49e9-9a41-7fe01c081262)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/b91f2a00-0070-4589-a5a6-4fcf27e05f09)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/9c622b33-1183-41b5-b930-e6ad97514694)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/04aed621-6110-44c5-9c91-14b3d48f0dfe)

We click on the repo and we upload the application docker image

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/67fd072b-daae-4e5b-9c57-0040e95d57c1)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/d515fcbe-c336-4d22-8bc9-352663afbb99)

These are the commnads we have to execute in VSCode Terminal Window

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/aa15477f-9757-42c1-9bd7-f6f16a8b345f)

```
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/x7p6e5r6
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/e379a11a-be44-4647-9e97-4618aa5541ff)

## 3. Create a Docker image and push it to Docker Desktop

This is the Dockerfile

```
# Start with a base image containing Java runtime
FROM openjdk:11-jdk-slim as build

# Add Maintainer Info
LABEL maintainer="your_email@example.com"

# Add a volume pointing to /tmp
VOLUME /tmp

# Make port 80 available to the world outside this container
EXPOSE 80

# The application's jar file
ARG JAR_FILE=target/demoapi-0.0.1-SNAPSHOT.jar

# Add the application's jar to the container
ADD ${JAR_FILE} demoapi.jar

# Run the jar file
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/demoapi.jar"]
```

Build the Docker image and push it to AWS ECR with these commands:

```
docker build -t springbootwebapirepo .

docker tag springbootwebapirepo:latest public.ecr.aws/x7p6e5r6/springbootwebapirepo:latest

docker push public.ecr.aws/x7p6e5r6/springbootwebapirepo:latest
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/98d7c677-ad5a-49ee-b9ec-95d938172748)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/63d6274f-1667-4c63-a45f-c4c5d119404e)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/50363885-ea21-4414-84b4-8d008c48fca1)

Run the Docker container with this command:

```
docker run -p 80:80 public.ecr.aws/x7p6e5r6/springbootwebapirepo:latest
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/f44caeb3-591f-4771-a13c-49d98c395710)

## 4. Create the AWS EKS cluster

We run this command for creating AWS EKS:

```
eksctl create cluster ^
--name springbootwebapi-cluster ^
--version 1.25 ^--region eu-west-3 ^
--nodegroup-name linux-nodes ^
--node-type t2.micro ^
--nodes 4
```

## 5. Deploy you application in AWS EKS Elastic Kluster 

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
  replicas: 1  # The number of Pods to run
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
          image: public.ecr.aws/x7p6e5r6/springbootwebapirepo:latest
          ports:
            - containerPort: 80
```

**IMPORTANT**: pay attention replace set the image name.

See section  3
...
image: public.ecr.aws/x7p6e5r6/springbootwebapirepo:latest
...

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
      targetPort: 80  # The port the container accepts traffic on
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

```
kubectl get service demoapi-service
```

```
kubectl get all
```

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/97d840ba-569d-4e17-804d-59bc0fce4073)

Use the external IP address to access your application in a web browser: **http://<EXTERNAL-IP>/hello**

http://ab80054383af243af94c3b39ab82add9-652326268.eu-west-3.elb.amazonaws.com/hello

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/344f90f9-d523-475e-97a5-66debe15b6d7)

![image](https://github.com/luiscoco/SpringBoot_Sample3-deploy_WebAPI-to-AWS_Kubernetes_EKS/assets/32194879/10948b78-74b0-4a47-aa6b-16903f22e05c)


