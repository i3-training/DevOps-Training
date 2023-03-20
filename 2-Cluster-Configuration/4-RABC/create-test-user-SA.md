## Create Service Account (SA)
1. Create name of service account
     ```sh
    kubectl create serviceaccount demo
    ```
2. Create token of service account
    ```sh
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: demo-secret
      annotations:
        kubernetes.io/service-account.name: demo
    type: kubernetes.io/service-account-token
    EOF
    ```
3. Describe secret value token SA
    ```sh
    TOKEN=$(kubectl describe secret demo-secret | grep token: | awk '{print $2}')
    echo $TOKEN
    ```
4. Configure service account
    ```sh
    kubectl config set-credentials demo --token=$TOKEN
    kubectl config set-context demo --cluster=kubernetes --user=imboyy
    kubectl config current-context
    kubectl config use-context demo
    ```
