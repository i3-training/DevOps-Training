Scale up and down manually with the kubectl scale command
Assume that today we'd like to scale our nginx Pods from two to four:

```sh
kubectl scale --replicas=<expected_replica_num> deployment <deployment_name>
```
example
```sh
kubectl scale --replicas=4 deployment my-nginxdeployment
```
