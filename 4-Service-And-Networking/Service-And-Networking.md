# Service And Networking

## 1. Install Calico
```
$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml​
$ curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml -O​

$ vi custom-resources.yaml​

...
spec:
  addresses:
  - 10.8.60.229-10.8.60.231
...

$ kubectl create -f custom-resources.yaml​
```

## 2. Create Deployment Apps
```
$ kubectl create deployment myapp --image=nginx --port=80

$ kubectl get deployments,pods
```

## 3. Expose Service ClusterIP
```
$ kubectl expose deployment/myapp
```

### URL of Service inside Kubernetes Cluster <service-name>.<namespace>.svc.cluster.local:<service-port>

## 4. Expose Service NodePort

### 4.1 via Run Command
```
$ kubectl expose deployment nginx --port 80 --type NodePort
```

### 4.2 via YAML
```
$ vi node-port-app.yml

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007

$ kubectl create -f node-port-app.yml
```

## 5. Expose Service LoadBalancer

### 5.1 Via Run Command
```
$ kubectl expose deploy myapp --port 80 --type LoadBalancer

```

### 5.2 Via YAML

```
$ vi lb-service.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
  namespace: prac-test
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: myapp
  type: LoadBalancer
status:
  loadBalancer: {}

$ kubectl get svc my-nginx
```

## 6. Install Ingress Controller
```
$ sudo dnf install helm 

$ helm repo add nginx-stable https://helm.nginx.com/stable​
$ helm repo update​
$ helm install ingress nginx-stable/nginx-ingress​
$ kubectl get svc ingress-nginx-ingress
```

## 7. Deploy Ingress 
```
$ vi my-ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  #namespace: <SERVICE-NAMESPACE>
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: <URL-TO-SERVICE-NAME>
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: <SERVICE-NAME>
                port:
                  number: <SERVICE-PORT>
    - host: <URL-TO-SERVICE-NAME>
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: <SERVICE-NAME>
                port:
                  number: <SERVICE-PORT>

$ kubectl apply -f my-ingress.yml
```