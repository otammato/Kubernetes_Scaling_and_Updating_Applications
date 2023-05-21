# Kubernetes_Scaling_and_Updating_Applications

- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Autoscale the application using Horizontal Pod Autoscaler

## Pre-steps:

1. Change to your project folder.

```
cd /home/project
``` 



2. Check if 'CC201' folder exists & clone the git repository that contains the files

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

```
kubectl scale deployment hello-world --replicas=3
```
2. Get Pods to ensure that there are now three Pods instead of just one. In addition, the status should eventually update to “Running” for all three.

```
kubectl get pods
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/e9eea591-a465-4400-ab6f-4bacff333c34" width="500px"/>
</p>

3. ```curl``` your application multiple times to ensure that Kubernetes is load-balancing across the replicas.
```bash
for i in `seq 10`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/e521605d-a3cf-48df-9e38-8db2e59320d2" width="500px"/>
</p>

You should see that the queries are going to different Pods because of the effect of load-balancing.

4. Similarly, you can use the ```scale``` command to scale down your Deployment.

```
kubectl scale deployment hello-world --replicas=1
```
5. Check the Pods to see that two are deleted or being deleted.

```
kubectl get pods
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/05ca8958-3418-4945-a3e4-06ee10a1a524" width="500px"/>
</p>

6. Wait for some time & run the same command again to ensure that only one pod exists.

```
kubectl get pods
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/8691fe97-00b1-4cff-bc57-701a720fab1d" width="500px"/>
</p>

<br>
<br>

## Perform rolling updates

Rolling updates are an easy way to update our application in an automated and controlled fashion. To simulate an update, let’s first build a new version of our application and push it to Container Registry.

1. Use the Explorer to edit ```app.js```. The path to this file is ```CC201/labs/3_K8sScaleAndUpdate/```. Change the welcome message from ```'Hello world from ' + hostname + '! Your app is up and running!\n'``` to ```'Welcome to ' + hostname + '! Your app is up and running!\n'```. Make sure to save the file when you’re done.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/fe7ad849-2f90-4a1f-9743-3974fdad1a7d" width="500px"/>
</p>

2. Build and push this new version to Container Registry. Update the tag to indicate that this is a second version of this application. Make sure to use the terminal window that isn’t running the ```proxy``` command.

```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:2 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:2
```
> NOTE: Do not close the terminal that is running the proxy command

```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:2 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:2
```

3. Update the deployment to use the new version.

```
kubectl set image deployment/hello-world hello-world=us.icr.io/$MY_NAMESPACE/hello-world:2
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/8e2f997b-18ee-4ce4-857f-571b74d7c34a" width="500px"/>
</p>

4. You can also get the Deployment with the wide option to see that the new tag is used for the image.

```
kubectl get deployments -o wide
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/0ebf1842-d497-43ce-93ae-045af652844d" width="500px"/>
</p>

5. Use ```curl``` command to ensure that the new welcome message is displayed.

```
curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/228df914-4f54-4e71-9760-84d61cf83897" width="500px"/>
</p>

6. It’s possible that a new version of an application contains a bug. In that case, Kubernetes can roll back the Deployment like this:
```
kubectl rollout undo deployment/hello-world
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/27473bc1-633c-4173-8d27-4894f4556d71" width="500px"/>
</p>

7. Get a status of the rolling update by using the following command:
```
kubectl rollout status deployment/hello-world
```
8. Get the Deployment with the wide option to see that the old tag is used.
```
kubectl get deployments -o wide
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/6fcdf302-fb36-4784-a451-6885204dd0a2" width="500px"/>
</p>

Look for the ```IMAGES``` column and ensure that the tag is ```1```.

9. Use ```curl``` command to ensure that the earlier ‘Hello World..Your app is up & running!‘ message is displayed.
```
curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/0d010fda-04a3-4278-ae81-9b39db2b15af" width="500px"/>
</p>

<br>
<br>

## Autoscale the ```hello-world``` application using Horizontal Pod Autoscaler
