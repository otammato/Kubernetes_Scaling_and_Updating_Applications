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
<br>

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


