helm repo add codecentric https://codecentric.github.io/helm-charts

kubectl create ns identity

helm upgrade --install keycloak codecentric/keycloak --values keycloak/values-keycloak.yml -n identity

