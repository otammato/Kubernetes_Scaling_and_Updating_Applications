# Kubernetes_Scaling_and_Updating_Applications

- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Use a ConfigMap to store application configuration
- Autoscale the application using Horizontal Pod Autoscaler

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


3. Build and push the image again, as it may have been deleted automatically since you completed the first lab.

```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
```

3. Deploy the application to Kubernetes
<br>
Use the Explorer to edit ` deployment.yaml ` in this directory. The path to this file is ```CC201/labs/3_K8sScaleAndUpdate/```. You need to insert your namespace where it says <my_namespace>. Make sure to save the file when you’re done.

```
```
