## Create Pod
The following is an example of a Pod which consists of a container running the image nginx:1.14.2.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
To create the Pod shown above, run the following command:

```sh
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```
Alternatively, you can create Pod using below command:
```sh
kubectl run <insert-pod-name-here> --image=nginx
```

or another example can create pod with name alpine.yaml 
```sh
apiVersion: v1
kind: Pod
metadata:
  name: alpine
spec:
  containers:
  - image: alpine
    name: alpine
    ##Create File
    # command: ["/bin/sh", "-c"]
    # args: ["shuf -i 0-100 -n 1 >> /opt/file.txt"]
    ##tty
    command: ["sh", "-c", "tail -f /dev/null"]
```
To create the Pod shown above, run the following command:
```sh
kubectl apply -f alpine.yaml
```
