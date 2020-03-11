# ASP.NET Core getting started application

The [Getting started tutorial for ASP.NET Core](https://cloud.ibm.com/docs/cloud-foundry?topic=cloud-foundry-getting_started-dotnet) uses this sample application to provide you with a sample workflow for working with any .NET Core app on IBM Cloud. In the workflow, you set up a development environment, deploy an app locally and on IBM Cloud, and integrate a IBM Cloud database service in your app. For detailed instructions, see the tutorial.

The ASP.NET Core getting started application uses a [Cloudant service](https://console.bluemix.net/catalog/services/cloudant) from IBM Cloud to add information to a database and then return information from the database to the application UI. 

<p align="center">
  <img src="https://raw.githubusercontent.com/IBM-Bluemix/get-started-java/master/docs/GettingStarted.gif" width="300" alt="Gif of the sample app contains a title that says, Welcome, a prompt asking the user to enter their name, and a list of the database contents which are the names Joe, Jane, and Bob. The user enters the name, Mary and the screen refreshes to display, Hello, Mary, I've added you to the database. The database contents listed are now Mary, Joe, Jane, and Bob.">
</p>


## IBM Cloud Commands

Command | Description
--- | --- 
ibmcloud login | Connect to IBM Cloud
ibmcloud target -r eu-de | Select a region such as eu-de
`ibmcloud target -g <resource-group-name>` | Select a Resource Group
ibmcloud plugin install kubernetes-service | Install the Kubernetes service plugin
ibmcloud plugin install container-registry | Install the Container Registry plugin

## Docker Commands

Command | Description
--- | --- 
`docker build . -t de.icr.io/<namespace/<image-name>:<image-tag>` | Build docker image
`docker run -d -p 8080:80 --name app de.icr.io/<namespace>/<image-name>:<image-tag>` | Run the docker container
`docker push de.icr.io/<namespace>/<image-name>:<image-tag>` | Push docker image on IBM Cloud


## IBM Cloud Kubernetes Service Plugin Commands

Command | Description
--- | --- 
`ibmcloud ks cluster config --cluster <cluster-name>` | Connect to cluster
`ibmcloud ks cluster pull-secret apply --cluster <cluster_name_or_ID>` | Create the secrets to connect to the Container Registry

## IBM Cloud Container Registry Plugin Commands

Command | Description
--- | --- 
ibmcloud cr login | Connect to the Container Registry 
ibmcloud cr namespaces | List existing namespaces
`ibmcloud cr namespace-add <new-namespace>` | Create a new namespace in the Container Registry
ibmcloud cr info | Identify your registry. For example: `de.icr.io`
ibmcloud cr image-list | Verify that the image was pushed

## Kubernetes Commands

Command | Description
--- | --- 
`kubectl create secret generic cloudant --from-literal=url=<URL>` | Create a Kubernetes secret
kubectl get secrets | List all the secrets
`kubectl apply -f <your-deployment.yaml>` | Deploy a container and its pods
`kubectl delete -f <your-deployment.yaml>` | Delete a deployment and its pods
`kubectl expose deployment <my-service> --type NodePort --port 80 --target-port 80` | Expose the service
kubectl get pods | List the pods
`kubectl describe po/<pod-instance-name>` | Get detailed info about a pod

## Dockefile

```
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2

WORKDIR /app1

COPY ./src/GetStartedDotnet/bin/Debug/netcoreapp2.2/publish/ .

ENTRYPOINT ["dotnet", "GetStartedDotnet.dll"]
```

## YAML

```
---
# Application to deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aspnetapp
spec:
  replicas: 2 # tells deployment to run 2 pods
  selector:
    matchLabels:
      app: aspnetapp
  template:   # create pods using pod definition in this template
    metadata:
      labels:
        app: aspnetapp
        tier: frontend
    spec:
      containers:
      - name: aspnetapp
        image: de.icr.io/cr-lma/aspnet:2.2
        imagePullPolicy: Always
        resources:
          requests:
            cpu: 250m     # 250 millicores = 1/4 core
            memory: 128Mi # 128 MB
          limits:
            cpu: 500m
            memory: 384Mi
        # envFrom:
        # - secretRef:
        #     name: cloudant
        #     optional: true
        env:
        - name: CLOUDANT_URL
          valueFrom:
            secretKeyRef:
              name: cloudant
              key: url
              optional: true
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: aspnetapp-ingress
  annotations:
    # Force the use of https if the request is http
    ingress.bluemix.net/redirect-to-https: "True"
spec:
  tls:
  - hosts:
    - mycluster-949312-16181e614890c5058ff637789396375b-0000.eu-de.containers.appdomain.cloud
    secretName: mycluster-949312-16181e614890c5058ff637789396375b-0000
  rules:
  - host: mycluster-949312-16181e614890c5058ff637789396375b-0000.eu-de.containers.appdomain.cloud
    http:
      paths:
      - path: /
        backend:
          serviceName: aspnetapp
          servicePort: 80
---
# Service to expose frontend
apiVersion: v1
kind: Service
metadata:
  name: aspnetapp
  labels:
    app: aspnetapp
    tier: frontend
spec:
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: aspnetapp
    tier: frontend
```

## Common errors

```
docker logs 3a1
Error:
  An assembly specified in the application dependencies manifest (GetStartedDotnet.deps.json) was not found:
    package: 'Microsoft.ApplicationInsights.AspNetCore', version: '2.1.1'
    path: 'lib/netstandard1.6/Microsoft.ApplicationInsights.AspNetCore.dll'
  This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
    aspnetcore-store-2.0.0-linux-x64.xml;aspnetcore-store-2.0.0-osx-x64.xml;aspnetcore-store-2.0.0-win7-x64.xml;aspnetcore-store-2.0.0-win7-x86.xml
```
Solution:
Add the following to .csproj:
```
<PropertyGroup>
  <PublishWithAspNetCoreTargetManifest>false</PublishWithAspNetCoreTargetManifest>
</PropertyGroup>
```


The ouput of the command `kubectl describe po/<pod-instance-name>` contains *Cannot pull image - Unauthorized*
Solution:
The secrets to connect to the container registry are missing somehow. You need to recreate those secrets in the cluster:
```
ibmcloud ks cluster pull-secret apply --cluster <cluster_name_or_ID>
```