# Goart Metaverse Case Study

The purpose of this documentation is to present a solution for a case study that has been developed specifically for the recruitment process of the GoArt Company.

## Prerequisites
- A local machine with Docker installed.
- A Kubernetes cluster with kubectl installed and configured.
- A DockerHub account.

## Clone the Sample-app

Clone the sample-app repository from GitHub:
```bash
git clone https://github.com/roofstacks/case-study-pool.git
```
## Dockerize the Sample-app


Create a Dockerfile in the ```~/case-study-pool/infrastructure-developer/mini-cluster/sample-app/``` directory of the sample-app project with the following content:

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "sample-app.dll"]
```
This Dockerfile uses two stages to build the app. In the first stage, it restores the dependencies and builds the app, and in the second stage, it copies the built app and runs it.


Build the Docker image with a tag name of your choice:
```bash
docker build -t metinekinci/goart-case:latest .
```
## Upload the Docker Image to DockerHub
Login to DockerHub with your DockerHub credentials:

```bash
docker login -u username
```

Push the Docker image to DockerHub:
```bash
docker push metinekinci/goart-case:latest
```

> **_NOTE:_**  The Docker image, which was generated using the following Dockerfile, has been pushed to Docker Hub and can be accessed via this link.
https://hub.docker.com/r/metinekinci/goart-case/tags

## Deploy the Sample-app on Kubernetes

Create a deployment with two replicas and use the Docker image you just uploaded:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: metinekinci/goart-case:latest
        ports:
        - containerPort: 80
```

Create a service for load balancing across the pods:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  selector:
    app: sample-app
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
Create an ingress for handling HTTP requests from outside the cluster:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-app
spec:
  ingressClassName: nginx
  rules:
  - host:
    http:
      paths:
      - path: /WeatherForecast
        pathType: Prefix
        backend:
          service:
            name: sample-app
            port:
              number: 80
```

After the following steps, the associated manifest file will look like the one in the [kube-manifest.yaml](./kube-manifest.yaml) file.

In order to apply the aforementioned manifest file, you will need to execute the subsequent command.

```bash
kubectl apply -f kube-manifest.yaml
```

You can verify the functionality of the application by sending a curl request to the URL provided below.

```bash
curl localhost/WeatherForecast
```

## One-click Install Script

You can create a one-click install script to automate the deployment process. Here is an example script:
```bash
#!/bin/bash

# Clone the repository
git clone https://github.com/metinekinci/goart_case.git

cd goart_case

# Build and push the Docker image to DockerHub
docker build -t <dockerhub-username>/goart-case:latest .
docker login -u <dockerhub-username>
docker push <dockerhub-username>/goart-case:latest

# Manipulate the manifest file
sed -i 's/metinekinci/<dockerhub-username>/g' kube-manifest.yaml

# Create a Kubernetes deployment and service
kubectl apply -f kube-manifest.yaml
```

This script performs the following steps:
- Clones the [goart_case](https://github.com/metinekinci/goart_case.git) repository.
- Builds and pushes the Docker image to DockerHub.
- The script manipulates the Kubernetes manifest file to make it available to the account of the person running the script.
- Applies the manifest file.

## Conclusion

This documentation explained how to Dockerize a .NET Core web app and deploy it on a Kubernetes cluster with an Ingress and a Service. It also included instructions for uploading the Docker image to DockerHub and creating a one-click install script.
