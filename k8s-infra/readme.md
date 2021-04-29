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

#Fetch Token and Cluster
TOKEN=$(kubectl get secrets $(kubectl get serviceaccounts $ACCOUNT -o jsonpath={.secrets[].name}) -o jsonpath={.data.token} | base64 --decode)
CLUSTER=$(kubectl config view --minify -o jsonpath='{.clusters[].name}')

#Modifing KUBECONFIG
kubectl config set-credentials $ACCOUNT --token=$TOKEN
kubectl config set-context $ACCOUNT-context --cluster $CLUSTER --user devuser
kubectl config use-context $ACCOUNT-context
```


# MetalLB
This is already done by deploying our Demo-Env
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"


cat << EOF > metallb-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.38.196.240-10.38.196.250 

EOF

kubectl apply -f metallb-config.yaml
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