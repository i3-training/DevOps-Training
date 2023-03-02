kubectl create serviceaccount imboyy

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: imboyy-secret
  annotations:
    kubernetes.io/service-account.name: imboyy
type: kubernetes.io/service-account-token
EOF

TOKEN=$(kubectl describe secret imboyy-secret | grep token: | awk '{print $2}')
echo $TOKEN

kubectl config set-credentials imboyy --token=$TOKEN
kubectl config set-context imboyy --cluster=kubernetes --user=imboyy
kubectl config current-context
kubectl config use-context imboyy
