1. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
   - Pod Name: messaging
   - Image: redis:alpine
   - Labels: tier=msg

2. Create a namespace named apx-x9984574.

3. Create a service messaging-service to expose the messaging application within the cluster on port 6379.
   - Service: messaging-service
   - Port: 6379
   - Type: ClusterIp
   - Use the right labels

4. Create a POD in the finance namespace named temp-bus with the image redis:alpine.
   - Name: temp-bus
   - Image Name: redis:alpine

5. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
   - Deployment : nginx-deploy. Image: nginx:1.16
   - Image: nginx:1.16
   - Task: Upgrade the version of the deployment to 1:17
   - Task: Record the changes for the image upgrade

6. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace .
   - Role Name: developer, namespace: development, Resource: Pods
   - Access: User 'john' has appropriate permissions

7. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
   Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.
   - ServiceAccount: pvviewer
   - ClusterRole: pvviewer-role
   - ClusterRoleBinding: pvviewer-role-binding
   - Pod: pvviewer
   - Pod configured to use ServiceAccount pvviewer ?

8. Taint the worker node node01 to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image: redis:alpine with toleration to be scheduled on node01.
   key: env_type, value: production, operator: Equal and effect: NoSchedule
   - Key = env_type
   - Value = production
   - Effect = NoSchedule
   - pod 'dev-redis' (no tolerations) is not scheduled on node01?
   - Create a pod 'prod-redis' to run on node01
