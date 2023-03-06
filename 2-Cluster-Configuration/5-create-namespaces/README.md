## Create Namespaces
1. Create a new YAML file called my-namespace.yaml with the contents:

```sh
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```

Then run:

```sh
kubectl create -f ./my-namespace.yaml
```

2. Alternatively, you can create namespace using below command:

```sh
kubectl create namespace <insert-namespace-name-here>
```

## Deleting a namespace 
Delete a namespace with

```sh
kubectl delete namespaces <insert-some-namespace-name>
```

## Viewing namespaces 
1. List the current namespaces in a cluster using:

```sh
kubectl get namespaces
```
```sh
NAME          STATUS    AGE
default       Active    11d
kube-system   Active    11d
kube-public   Active    11d
```
Or you can get detailed information with:
```sh
kubectl describe namespaces <name>
```
```sh
Name:           default
Labels:         <none>
Annotations:    <none>
Status:         Active

No resource quota.

Resource Limits
 Type       Resource    Min Max Default
 ----               --------    --- --- ---
 Container          cpu         -   -   100m
```

