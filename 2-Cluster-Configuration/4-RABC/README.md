## Implementing RBAC In Your Kubernetes Cluster
Most new clusters will start you with a fully privileged user account that can perform any Kubernetes action. RBAC is an optional feature that can be turned off altogether. Run the following command to see if it’s enabled:

```sh
$ kubectl api-versions | grep rbac.authorization.k8s
rbac.authorization.k8s.io/v1
```
The command above has produced a line of output which shows RBAC is available. If you don’t see any output, that indicates that RBAC is turned off in your cluster. You can enable the feature by starting the Kubernetes API server with the RBAC authorization mode included:

```sh
$ kube-apiserver --authorization-mode=RBAC
```
## Creating a Test User
Next, you’ll create a test user account that you’ll assign your RBAC roles to. Kubernetes recognizes two types of user: a normal user and a service account. Normal users can’t be added using regular API calls, so we’ll create a service account for this tutorial. The Kubernetes documentation includes a description of the differences between the user types and explanations of when each one is appropriate.

Run the following command to add a new service account called demo:

```sh
$ kubectl create serviceaccount demo
serviceaccount/demo created
```
Find the name of the secret that stores the service account’s token:

```sh
$ kubectl describe serviceaccount demo
Name:               demo
Namespace:          default
Labels:             <none>
Annotations:        <none>
Image pull secrets:  <none>
Mountable secrets:   demo-token-znwmb
Tokens:             demo-token-znwmb
Events:             <none>
```
The secret’s name is demo-token-znwmb. Retrieve the token’s value from the secret using the following command:

```sh
$ TOKEN=$(kubectl describe secret demo-token-znwmb | grep token: | awk '{print $2}')
$ echo $TOKEN
eyJbGciOi...
```
With the token extracted, you can add your user as a new kubectl context:

```sh
$ kubectl config set-credentials demo --token=$TOKEN
User "demo" set.
```
```sh
$ kubectl config set-context demo --cluster=kubernetes --user=demo
Context "demo" created.
```
If your current cluster connection is not called kubernetes, you should change the value of the --clusterflag.

Now you can switch to your demo context to run kubectl commands as your new service account user. Run the current-context command first to check the name of your currently selected context—you’ll be switching back to it in a moment.

```sh
$ kubectl config current-context
default
```
```sh
$ kubectl config use-context demo
Switched to context "demo".
```
Try to retrieve the pods in your cluster:

```sh
$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:demo" cannot list resource "pods" in API group "" in the namespace "default"
```
Kubernetes has issued a Forbidden error. The demo service account has been recognized, but because no roles have been created or assigned, it lacks permission to run the get pods command.

## Creating a Role
Switch back to your original context before continuing, so you regain your administrative privileges:

```sh
$ kubectl config use-context default
Switched to context "default".
```
Kubernetes role objects are created in the standard way. You write a YAML file that defines the role’s rules, then use kubectl to add it to your cluster.

To keep things simple, you’ll use the “Developer” role example from earlier:

```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: Developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "get", "list"]
```
Users who are assigned this role will be able to run the create pod, get pod, get pods, and describe pod commands. They won’t be able to update or delete pods, or perform actions that target other resource types.

Use kubectl to add the role to your cluster:

```sh
$ kubectl apply -f role.yaml
role.rbac.authorization.k8s.io/Developer created
```

## Creating a RoleBinding
A role on its own has no effect on your cluster’s access control. It needs to be bound to users with a RoleBinding. Create this object now to grant your demo service account the permissions associated with your Developer role:

```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: DeveloperRoleBinding
subjects:
  - kind: ServiceAccount
    name: demo
    apiGroup: ""
roleRef:
  kind: Role
  name: Developer
  apiGroup: ""
```
The subjects field lists the objects that will be granted the permissions included in the role. In this example, you’re adding a single ServiceAccount subject to represent your demo user. You can target a User or Group instead by adjusting the subject’s Kind accordingly.

The roleRef field identifies the role that will be bound to the subjects. The kind can be either Role or ClusterRole, depending on the type of role you’ve created. In this example, the Developer role is a regular role.

Add the RoleBinding to your cluster using kubectl:

```sh
$ kubectl apply -f role-binding.yaml
rolebinding.rbac.authorization.k8s.io/DeveloperRoleBinding created
```
Testing Your RBAC Policy
The RoleBinding should have granted your demo service account the permissions included in the Developer role. Verify this by switching back to your demo context:

```sh
$ kubectl config use-context demo
Switched to context "demo".
```
Repeat the get pods command:

```sh
$ kubectl get pods
No resources found in default namespace.
```
This time the command succeeds. The get pods operation is permitted by the Developer role, which is now assigned to the demo service account you’re authenticated as.

Create a new pod:

```sh
$ kubectl run nginx --image=nginx
pod/nginx created
```
This operation is permitted, too. Now, try to delete the superfluous pod:

```sh
$ kubectl delete pod nginx
Error from server (Forbidden): pods "nginx" is forbidden: User "system:serviceaccount:default:demo" cannot delete resource "pods" in API group "" in the namespace "default"
```
Kubernetes rejects the operation because the demo service account doesn’t have a role that provides a suitable permission. This demonstrates how you can use Role and RoleBinding objects to restrict how users interact with your cluster.
