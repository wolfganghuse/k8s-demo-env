# Create Network Infrastructure Service (Load Balancer / Ingress / Cert-Manager)

# Create persistent Service Account
This is already done by deploying our Demo-Env

```
ACCOUNT=devuser

#Create SA
kubectl create sa $ACCOUNT

#Create ClusterRoleBinding
cat << EOF > $ACCOUNT-rb.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: $ACCOUNT-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: $ACCOUNT
  namespace: default
EOF

kubectl create -f $ACCOUNT-rb.yaml

#Create Token and Cluster

kubectl create -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: $ACCOUNT-secret
  annotations:
    kubernetes.io/service-account.name: $ACCOUNT
type: kubernetes.io/service-account-token
EOF

TOKEN=$(kubectl get secrets $ACCOUNT-secret -o jsonpath={.data.token} | base64 --decode)
CLUSTER=$(kubectl config view --minify -o jsonpath='{.clusters[].name}')

#Modifing KUBECONFIG
kubectl config set-credentials $ACCOUNT --token=$TOKEN
kubectl config set-context $ACCOUNT-context --cluster $CLUSTER --user devuser
kubectl config use-context $ACCOUNT-context
```


# MetalLB
This is already done by deploying our Demo-Env
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml


kubectl apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: cheap
  namespace: metallb-system
spec:
  addresses:
  - 10.48.38.41-10.48.38.42
EOF
```

# Ingress
```
kubectl create ns ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install quickstart ingress-nginx/ingress-nginx --namespace ingress-nginx
```

# Cert-Manager
```
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.3.1 --set installCRDs=true
```