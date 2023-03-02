kubectl scale Use Cases
The kubectl scale command is used to change the number of running replicas inside Kubernetes deployment, replica set, replication controller, and stateful set objects. When you increase the replica count, Kubernetes will start new pods to scale up your service. Lowering the replica count will cause Kubernetes to gracefully terminate some pods, freeing up cluster resources.

You can run kubectl scale to manually adjust your application’s replica count in response to changing service capacity requirements. Increased traffic loads can be handled by increasing the replica count, providing more application instances to serve user traffic. When the surge subsides, the number of replicas can be reduced. This helps keep your costs low by avoiding utilization of unneeded resources.

Using kubectl
The most basic usage of kubectl scale looks like this:

```sh
$ kubectl scale --replicas=3 deployment/demo-deployment
Executing this command will adjust the deployment called demo-deployment so it has three running replicas. You can target a different kind of resource by substituting its name instead of deployment:
```

# ReplicaSet
```sh
$ kubectl scale --replicas=3 rs/demo-replicaset
```

# ReplicationController
```sh
$ kubectl scale --replicas=3 rc/demo-replicationcontroller
```

# StatefulSet
```sh
$ kubectl scale --replicas=3 sts/demo-statefulset
```
Basic Scaling
Now we’ll look at a complete example of using kubectl scale to scale a deployment. Here’s a YAML file defining a simple deployment:

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest

```
Save this YAML to demo-deployment.yaml in your working directory. Next, use kubectl to add the deployment to your cluster:

```sh
$ kubectl apply -f demo-deployment.yaml
deployment.apps/demo-deployment created
```
Now run the get pods command to view the pods that have been created for the deployment:
```sh
$ kubectl get pods

NAME	READY	STATUS	RESTARTS	AGE
demo-deployment-86897ddbb-jl6r6	1/1	Running	0	33s
```
Only one pod is running. This is expected, as the deployment’s manifest declares one replica in its spec.replicas field.

A single replica isn’t sufficient for a production application. You could experience downtime if the node hosting the pod goes offline for any reason. Use kubectl scale to increase the replica count to provide more headroom:

```sh
$ kubectl scale --replicas=5 deployment/demo-deployment
deployment.apps/demo-deployment scaled
```
Repeat the get pods command to confirm that the deployment has been scaled successfully:
```sh
$ kubectl get pods

NAME	READY	STATUS	RESTARTS	AGE
demo-deployment-86897ddbb-66lzc	1/1	Running	0	46s
demo-deployment-86897ddbb-66s9d	1/1	Running	0	46s
demo-deployment-86897ddbb-jl6r6	1/1	Running	0	3m33s
demo-deployment-86897ddbb-sgcjb	1/1	Running	0	46s
demo-deployment-86897ddbb-tgvnw	1/1	Running	0	46s
```
There are now five pods running for the demo-deployment deployment. You can see from the AGE column that the scale command retained the original pod and added four new ones.

After further consideration, you might decide five replicas are unnecessary for this application. It’s only running a static NGINX web server, so resource consumption per user request should be low. Use the scale command again to lower the replica count and avoid wasting cluster capacity:

```sh
$ kubectl scale --replicas=3 deployment/demo-deployment
deployment.apps/demo-deployment created
```
Repeat the get pods command:
```sh
$ kubectl get pods

NAME	READY	STATUS	RESTARTS	AGE
demo-deployment-86897ddbb-66lzc	1/1	Terminating	0	3m21s
demo-deployment-86897ddbb-66s9d	1/1	Terminating	0	3m21s
demo-deployment-86897ddbb-jl6r6	1/1	Running	0	6m8s
demo-deployment-86897ddbb-sgcjb	1/1	Running	0	3m21s
demo-deployment-86897ddbb-tgvnw	1/1	Running	0	3m21s
```
Kubernetes has marked two of the running pods for termination. This will reduce the running replica count down to the requested three pods. The pods selected for eviction are sent a SIGTERM signal and allowed to gracefully terminate. They’ll be removed from the pod list once they’ve stopped.


Monitor Kubernetes Events in Real-Time
Monitor the health of your cluster and troubleshoot issues faster with pre-built dashboards that just work.
Learn More
event dashboard

Conditional Scaling
Sometimes you might want to scale a resource, but only if there’s a specific number of replicas already running. This avoids unintentional overwrites of previous scaling changes, such as those made by other users in your cluster.

Include the --current-replicas flag in the command to use this behavior:

```sh
$ kubectl scale --current-replicas=3 --replicas=5 deployment/demo-deployment
deployment.apps/demo-deployment scaled
```
This example scales the demo-deployment deployment to five replicas, but only if there’s currently three replicas running. The --current-replicas value is always matched exactly; you can’t express a condition as “less than” or “greater than” a particular count.

Scaling Multiple Resources
The kubectl scale command can scale several resources at once when you supply more than one name as arguments. Each of the resources will be scaled to the same replica count set by the --replicas flag.

```sh 
$ kubectl scale --replicas=5 deployment/app deployment/database
deployment.apps/app scaled
deployment.apps/database scaled
```
This command scales the app and database deployments to five replicas each.

You can scale every resource of a particular type by supplying the --all flag, such as this example to scale all the deployments in your default namespace:

```sh
$ kubectl scale --all --replicas=5 --namespace=default deployment
deployment.apps/app scaled
deployment.apps/database scaled
```
This selects every matching resource inside the currently active namespace. The objects that were scaled are shown in the command’s output.

You can obtain granular control over the objects that are scaled with the --selector flag. This lets you use standard selection syntax to filter objects based on their labels. Here’s an example that scales all the deployments with an app-name=demo-app label:

```sh
$ kubectl scale --replicas=5 --selector=app-name=demo-app deployment
deployment.apps/app scaled
deployment.apps/database scaled
```
