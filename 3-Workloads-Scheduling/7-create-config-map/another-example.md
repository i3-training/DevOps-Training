The simplest way to create a ConfigMap is to store a bunch of key-value strings in a ConfigMap YAML file and inject them as environment variables into your Pods.

Creating a ConfigMap
This YAML creates a ConfigMap with the value database set to mongodb, and database_uri, and keys set to the values in the YAML example code.

Then, create the ConfigMap in the cluster using kubectl apply -f config-map.yaml.
```sh
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # Configuration values can be set as key-value properties
  database: mongodb
  database_uri: mongodb://localhost:27017
  
  # Or set as complete file contents (even JSON!)
  keys: | 
    image.public.key=771 
    rsa.public.key=42
```
Using a ConfigMap in Environment Variables
The key to adding your ConfigMap as environment variables to your pods is the envFrom property in your Pod’s YAML.

Set envFrom to a reference to the ConfigMap you’ve created.
```sh
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:1.7.9 
      envFrom:
        - configMapRef:
            name: example-configmap
```
After you’ve created the Pod, you’ll be able to access these environment variables.
```sh
$ kubectl apply -f pod-env-var.yaml
pod "pod-env-var" created

$ kubectl exec -it pod-env-var sh
# env
# ...
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
database=mongodb
HOSTNAME=pod-env-var
database_uri=mongodb://localhost:27017
HOME=/root
keys=image.public.key=771 
rsa.public.key=42

TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1.7.9-1~wheezy
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
```

