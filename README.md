# Kubernetes_Scaling_and_Updating_Applications

- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Use a ConfigMap to store application configuration
- Autoscale the application using Horizontal Pod Autoscaler

## Pre-steps:

1. Change to your project folder.

```
cd /home/project
``` 



2. Clone the git repository that contains the files

```
[ ! -d 'CC201' ] && git clone https://github.com/ibm-developer-skills-network/CC201.git
```
3. Change to the directory

```
cd CC201/labs/3_K8sScaleAndUpdate/
```
<br>
<br>

## Build and push application image to Cloud Container Registry
1. Export your namespace as an environment variable so that it can be used in subsequent commands.
```
export MY_NAMESPACE=sn-labs-$USERNAME
```
2. Use the Explorer to view the Dockerfile that will be used to build an image.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/459a71fd-42f9-48e5-ab91-f380391aa19b)" width="500px"/>
</p>


3. Build and push the image.

```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
```

<br>
<br>

## Deploy the application to Kubernetes

1. Use the Explorer to edit ```deployment.yaml``` in this directory. The path to this file is ```CC201/labs/3_K8sScaleAndUpdate/```. You need to insert your namespace where it says <my_namespace>. Make sure to save the file when you’re done.

> NOTE: To know your namespace, run ```echo $MY_NAMESPACE``` in the terminal

2. Run your image as a Deployment.

```yml
kubectl apply -f deployment.yaml
```
3. List Pods until the status is “Running”.

```
kubectl get pods
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/3431f65d-4dcd-48f1-892d-91fd9b0d5f26" width="500px"/>
</p>

4. In order to access the application, we have to expose it to the internet via a Kubernetes Service.

```
kubectl expose deployment/hello-world
```
This creates a service of type ClusterIP.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/b94467ea-18ad-4339-9025-0d746e5cc6d5" width="500px"/>
</p>

5. Open a new terminal window using Terminal > New Terminal

> NOTE: Do not close the terminal window you were working on.

6. Currently, cluster IPs are only accesible within the cluster. To make this externally accessible, we will create a proxy.

```
kubectl proxy
```
> Note: This is not how you would make an application externally accessible in a production scenario.

7. Go back to your original terminal window, ping the application to get a response.

```
curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/a3f1ae19-89f1-4e8a-9072-c648123ccbdb" width="500px"/>
</p>

The app is running!

<br>
<br>

## Scaling the application using a ReplicaSet

In real-world situations, load on an application can vary over time. If our application begins experiencing heightened load, we want to scale it up to accommodate that load. There is a simple ```kubectl``` command for scaling.

1. Use the ```scale``` command to scale up your Deployment. Make sure to run this in the terminal window that is not running the ```proxy``` command.

```py
kubectl scale deployment hello-world --replicas=3
```
2. Get Pods to ensure that there are now three Pods instead of just one. In addition, the status should eventually update to “Running” for all three.
