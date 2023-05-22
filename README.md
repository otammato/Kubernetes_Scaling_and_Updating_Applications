# Kubernetes_Scaling_and_Updating_Applications

- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Autoscale the application using Horizontal Pod Autoscaler

## Pre-steps:
0. Deploy a Kubernetes cluster. You can use any of this approaches:
https://github.com/otammato/Kubernetes_deploy_cluster_manually_on_EC2_instances_AWS
https://github.com/otammato/Kubernetes_Terraform_cluster_deploy_on_EKS_AWS
https://github.com/otammato/Kubernetes_Terraform_deploy_on_EC2_AWS

1. Change to your project folder.

```
cd /home/project
``` 


2. Check if 'CC201' folder exists & clone the git repository that contains the files

```
[ ! -d 'CC201' ] && git clone https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications.git
```
3. Change to the directory

```
cd CC201/labs/3_K8sScaleAndUpdate/
```
<br>
<br>

## Build and push application image to a Cloud Container Registry (I'm using IBM ICR here, can be Docker Hub or AWS ECR)
1. Export your namespace as an environment variable so that it can be used in subsequent commands.
```
export MY_NAMESPACE=sn-labs-$USERNAME
```
2. Use the Explorer to view the Dockerfile that will be used to build an image.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/031db1d5-06a3-4777-befa-5f3b939e7262" width="600px"/>
</p>


3. Build and push the image.

```
docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
```

<br>
<br>

## Deploy the application to Kubernetes

1. Use the Explorer to edit ```deployment.yaml``` in this directory. The path to this file is ```CC201/labs/3_K8sScaleAndUpdate/```. You need to insert your namespace where it says <my_namespace>. Make sure to save the file when you’re done.

> NOTE: To know your namespace, run ```echo $MY_NAMESPACE``` or ```env | grep MY_NAMESPACE``` in the terminal 

2. Run your image as a Deployment.

```yml
kubectl apply -f deployment.yaml
```
3. List Pods until the status is “Running”.

```
kubectl get pods
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/4fb2df96-3e1d-4ff0-b458-1b8c4eea0d03" width="800px"/>
</p>

4. In order to access the application, we have to expose it to the internet via a Kubernetes Service.

```
kubectl expose deployment/hello-world
```
This creates a service of type ClusterIP.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/f63e878c-9c30-41c9-b135-0245aed84230" width="800px"/>
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
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/0a4ab8d2-c856-4645-8efd-6309e80ad703" width="800px"/>
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
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/af376e0a-8092-408d-a532-148994f42911" width="800px"/>
</p>

3. ```curl``` your application multiple times to ensure that Kubernetes is load-balancing across the replicas.
```bash
for i in `seq 10`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/748aed8a-0a19-43a9-b86f-8483463fbdc3" width="800px"/>
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
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/c0baf046-d51e-451b-b6b9-c61bfecf6fa7" width="800px"/>
</p>

6. Wait for some time & run the same command again to ensure that only one pod exists.

```
kubectl get pods
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/ae322a51-7b4d-45cc-af6c-dcd403162e3b" width="800px"/>
</p>

<br>
<br>

## Perform rolling updates

Rolling updates are an easy way to update our application in an automated and controlled fashion. To simulate an update, let’s first build a new version of our application and push it to Container Registry.

1. Use the Explorer to edit ```app.js```. The path to this file is ```CC201/labs/3_K8sScaleAndUpdate/```. Change the welcome message from ```'Hello world from ' + hostname + '! Your app is up and running!\n'``` to ```'Welcome to ' + hostname + '! Your app is up and running!\n'```. Make sure to save the file when you’re done.

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/a3442360-9511-4629-abbf-20a8f66c461f" width="800px"/>
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
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/f1e7a7c2-6b69-4c79-891c-35528120cf15" width="800px"/>
</p>

4. You can also get the Deployment with the wide option to see that the new tag is used for the image.

```
kubectl get deployments -o wide
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/29e06fa7-f384-44e1-bc4d-bcd3ebdaa008" width="800px"/>
</p>


5. Use ```curl``` command to ensure that the new welcome message is displayed.

```
curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/09cf5878-5b54-49f4-beb8-c05d01bfc33c" width="800px"/>
</p>


6. It’s possible that a new version of an application contains a bug. In that case, Kubernetes can roll back the Deployment like this:
```
kubectl rollout undo deployment/hello-world
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/ed7583d7-1a40-42bc-9701-41d154695a06" width="800px"/>
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
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/b820c4fb-d225-4f9a-a516-b4a0be68f43b
" width="800px"/>
</p>

Look for the ```IMAGES``` column and ensure that the tag is ```1```.

9. Use ```curl``` command to ensure that the earlier ‘Hello World..Your app is up & running!‘ message is displayed.
```
curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/1a9b90bb-3145-45bc-a2f1-805db34a461c" width="800px"/>
</p>

<br>
<br>

## Autoscale the ```hello-world``` application using Horizontal Pod Autoscaler

1. Add the following section to the deployment.yaml file under the template.spec.containers section
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/954b2472-afe3-4c3b-82d3-eb3a45829cc8" width="600px"/>
</p>

The added into the YAML manifest section specifies the CPU resource limits and requests for the container within the hello-world deployment.
resources specifies the resource requirements for the container.

- ```limits``` define the maximum amount of CPU resources the container can use.
- ```cpu: 50m``` sets the CPU limit to 50 milliCPU units. The "m" suffix represents milliCPU units, where 1 CPU unit is equivalent to 1000 milliCPU units.
- ```requests``` define the minimum amount of CPU resources the container needs to run.
- ```cpu:``` 20m sets the CPU request to 20 milliCPU units.

In this configuration, the container is limited to a maximum of 50 milliCPU units (0.05 CPU units) and requires at least 20 milliCPU units (0.02 CPU units) to be scheduled on a node.

2. Apply the deployment:

```
kubectl apply -f deployment.yaml
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/f872aced-7245-47e5-a4a3-223b3adc1ff9" width="600px"/>
</p>

3. Autoscale the ```hello-world``` deployment using the below command:

```
kubectl autoscale deployment hello-world --cpu-percent=5 --min=1 --max=10
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/6e3ebbca-04ad-440b-b276-3941efcaf99d" width="600px"/>
</p>

By running this command, you enable horizontal pod autoscaling for the "hello-world" deployment, with a target CPU utilization of 5%. The number of replicas will dynamically scale between 1 and 10 based on the CPU utilization.

4. You can check the current status of the newly-made HorizontalPodAutoscaler, by running:


```
kubectl get hpa hello-world
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/c132bcb1-f052-4467-918a-c45b7c8311bd" width="600px"/>
</p>

5. Please ensure that the kubernetes proxy is still running in the 2nd terminal. If it is not, please start it again by running:

```
kubectl proxy
```

6. Open another new terminal and enter the below command to spam the app with multiple requests for increasing the load:

```
for i in `seq 100000`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
```

This is a bash loop that repeatedly sends a curl request to the specified URL. It will execute the curl command 100,000 times in a loop.

7. Run the below command to observe the replicas increase in accordance with the autoscaling:
```
kubectl get hpa hello-world --watch
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/d099b715-663a-46c6-9102-aa6a1618b499" width="600px"/>
</p>

You will see an increase in the number of replicas which shows that your application has been autoscaled.
<br>
Stop this command by pressing ```CTRL + C```

8. Delete the Deployment.

```
kubectl delete deployment hello-world
```
<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/b3e73a44-b7d5-4865-a534-c60c6d2381ea" width="600px"/>
</p>

9. Delete the Service.

```
kubectl delete service hello-world
```

<p align="left" >
  <img src="https://github.com/otammato/Kubernetes_Scaling_and_Updating_Applications/assets/104728608/be531d61-9f8a-4bec-8930-46ef91ab8225" width="600px"/>
</p>

