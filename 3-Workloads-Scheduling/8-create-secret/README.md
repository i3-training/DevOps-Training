## Solution
Before you can get started using secrets, you first need to create a secret. As you may expect, this can be done by defining an object of kind Secret, name it mysecret.yaml:
```sh
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: bXl1c2VybmFtZQo= #Base64 encoded value of "myusername"
  password: bXlwYXNzd29yZAo= #Base64 encoded value of "mypassword"
```
Secrets in Kubernetes are, at their most basic form, a collection of keys and values. The above example creates a secret named mysecret with two keys: username and password. There’s one very important thing to note though, which is that the values of these key/value pairs are encoded as base64. Remember that base64 is an encoding algorithm, not an encryption algorithm. This is done to help facilitate data that may not be entirely alpha-numeric, and instead could include binary data, non-ASCII data, etc.

Once applied, you can see that while you can get the secret with kubectl, it avoids printing the values of each key by default:
```sh
$ kubectl describe secret mysecret

Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  
Type:         Opaque

Data
====
password:  11 bytes
username:  11 bytes
```
Of course, if you want to see the base64-encoded contents of the secret, you can still fetch them with a slightly different command:
```sh
$ kubectl get secret mysecret -o yaml

apiVersion: v1
data:
  password: bXlwYXNzd29yZAo=
  username: bXl1c2VybmFtZQo=
kind: Secret
...
```
Great! With your secret created, it’s time to start creating pods to use it! You’re faced with another decision, however, since Kubernetes provides a couple of methods for presenting secrets to a pod. The first example you’ll look at is mounting them as files in a volume:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: secret-as-file
spec:
  containers:
  - name: secret-as-file
    image: nginx
    volumeMounts:
    - name: mysecretvol
      mountPath: "/etc/mysecret"
      readOnly: true
  volumes:
  - name: mysecretvol
    secret:
      secretName: mysecret
```
Here, a new pod named secret-as-file is created from the NGINX Docker image.

NOTE: The nginx container image is used here simply because it’s an easily accessible long-running process, this would look the same for your own container image.

There are two sections to point out, the first being the volumes section, which defines a new volume. Kubernetes has many different types of volumes to choose from, but for this case you’re specifically interested in creating a volume of type secret. These volumes are backed by tmpfs, a RAM-based file system, rather than written to a persistent disk. Secret volumes require you to define the secret to mount (in the secretName field), and for each key in your secret, it creates a file that contains the key’s value. You can see this in action by applying this YAML and then listing the files at the mountPath:
```sh
https://raw.githubusercontent.com/i3-training/DevOps-Training/main/3-Workloads-Scheduling/8-create-secret/create-secret/secret/secret/pod-secret-file.yaml
```
```sh
$ kubectl exec secret-as-file -- ls /etc/mysecret

password
username

$ kubectl exec secret-as-file -- cat /etc/mysecret/username

myusername
```
As you can see, there are two files in the volume that was created: password and username. If you print out the contents of the username file, you can see the secret’s value of myusername.
```sh
$ kubectl exec secret-as-file -- cat /etc/mysecret/username

myusername
```
Alternatively, secrets can also be presented to your container as environment variables. Consider the following YAML:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: secret-as-env
spec:
  containers:
  - name: secret-as-env
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```
Here, instead of defining volumes that reference your secret, two environment variables are defined and reference the secret name and key name. Applying this YAML allows you to retrieve these environment variables from a shell in the pod:
```sh
https://raw.githubusercontent.com/i3-training/DevOps-Training/main/3-Workloads-Scheduling/8-create-secret/create-secret/secret/secret/pod-secret-env.yaml
```
```sh
$ kubectl exec secret-as-env -- sh -c "echo \$SECRET_USERNAME"

myusername
```
As you can see, the value of your secret is stored in the $SECRET_USERNAME environment variable as defined!
