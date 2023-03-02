# Storage

## Volumes

### 1. EmptyDir

```
$ kubectl create -f emptyDir.yml

$ kubectl get pod

$ kubectl exec -it test-pd -n prac-oprek -- touch /cache/1.txt

$ kubectl exec -it test-pd -n prac-oprek -- ls /cache

$ ls /var/lib/kubelet/pods/{podid}/volumes/kubernetes.io~empty-dir/
```

### 2. HostPath
```
$ kubectl create -f hostPath.yml

$ kubectl get pod

$ kubectl exec -it test-webserver -n prac-oprek -- sh
$ ls /var/local/aaa
$ exit

to worker node
$ ls /var/local/aaa

$ kubectl delete -f hostPath.yml
$ ls /var/local/aaa
```

### 3. PersistentVolume and PersistentVolumeClaim
#### Static Provisioning (HostPath)
```
$ kubectl create -f pv.yml

$ kubectl create -f pvc.yml

$ kubectl create -f pod-storage.yml

$ kubectl get pod
$ kubectl get pvc
$ kubectl get pv

$ ls /mnt/data
$ kubectl delete -f pod-storage.yml
$ ls /mnt/data
```
#### Dynamic Provisioning
```
$ vi pvc.yml

...
spec:
  ...
  storageClassName: nfs
  ...
...

$ kubectl create -f pvc.yml
$ kubectl create -f pod-storage.yml

$ kubectl get pvc
$ kubectl get pv
$ kubectl describe pv <PV-apps> | grep Path

to nfs server
$ ls /<path-to-nfs-dir>/<pv-path>

$ kubectl delete -f pod-storage.yml
$ ls /<path-to-nfs-dir>/<pv-path>

```

$ kubectl get pods -n <namespace> <pod-name> -o jsonpath='{.metadata.uid}'

```

