1. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
   - Pod Name: messaging
   - Image: redis:alpine
   - Labels: tier=msg

2. Create a namespace named apx-x9984574.

1. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
   - Deployment : nginx-deploy. Image: nginx:1.16
   - Image: nginx:1.16
   - Task: Upgrade the version of the deployment to 1:17
   - Task: Record the changes for the image upgrade

   Run the below command for solution:

     <details>
 
     For Kubernetes Version <=1.17
 
     ```
     kubectl run nginx-deploy --image=nginx:1.16 --replicas=1 --record
     kubectl rollout history deployment nginx-deploy
     kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
     kubectl rollout history deployment nginx-deploy
     ```
 
     For Kubernetes Version >1.17
 
     ```
     kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml
   
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deploy
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: nginx-deploy
       strategy: {}
       template:
         metadata:
           creationTimestamp: null
           labels:
             app: nginx-deploy
         spec:
           containers:
           - image: nginx:1.16
             name: nginx
     ```
     
     ```
     kubectl create -f deploy.yaml --record
     kubectl rollout history deployment nginx-deploy
     kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
     kubectl rollout history deployment nginx-deploy
     ```
     </details>

2. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace .
   - Role Name: developer, namespace: development, Resource: Pods
   - Access: User 'john' has appropriate permissions

3. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
   Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.
   - ServiceAccount: pvviewer
   - ClusterRole: pvviewer-role
   - ClusterRoleBinding: pvviewer-role-binding
   - Pod: pvviewer
   - Pod configured to use ServiceAccount pvviewer ?
  
   Run the below command for solution: 

     <details>

     ```
     kubectl create serviceaccount pvviewer
     kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
     kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
     ```

     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       labels:
         run: pvviewer
       name: pvviewer
     spec:
       containers:
       - image: redis
         name: pvviewer
         resources: {}
       serviceAccountName: pvviewer
     ```
     </details>

4. Taint the worker node node01 to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image: redis:alpine with toleration to be scheduled on node01.
   key: env_type, value: production, operator: Equal and effect: NoSchedule
   - Key = env_type
   - Value = production
   - Effect = NoSchedule
   - pod 'dev-redis' (no tolerations) is not scheduled on node01?
   - Create a pod 'prod-redis' to run on node01

   Run the below command for solution: 
 
     <details>
 
     ```
     kubectl taint node node01 env_type=production:NoSchedule
     ```

     Deploy `dev-redis` pod and to ensure that workloads are not scheduled to this `node01` worker node.
     ```
     kubectl run dev-redis --image=redis:alpine

     kubectl get pods -owide
     ```

     Deploy new pod `prod-redis` with toleration to be scheduled on `node01` worker node.
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       name: prod-redis
     spec:
       containers:
       - name: prod-redis
         image: redis:alpine
       tolerations:
       - effect: NoSchedule
         key: env_type
         operator: Equal
         value: production     
     ```

     View the pods with short details: 
     ```
     kubectl get pods -owide | grep prod-redis
     ```
     </details>
