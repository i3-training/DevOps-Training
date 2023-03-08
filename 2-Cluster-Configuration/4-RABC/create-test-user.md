```sh
kubectl create serviceaccount student

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: student-secret
  annotations:
    kubernetes.io/service-account.name: student
type: kubernetes.io/service-account-token
EOF
```
```sh
TOKEN=$(kubectl describe secret imboyy-secret | grep token: | awk '{print $2}')
echo $TOKEN
```
```sh
kubectl config set-credentials student --token=$TOKEN
kubectl config set-context student --cluster=kubernetes --user=imboyy
kubectl config current-context
kubectl config use-context student
```
