# Persistent data Kubernetes

```
cd .\persistentvolume\

kubectl create ns postgres


kubectl apply -f persistentvolume.yaml
kubectl apply -n postgres -f persistentvolumeclaim.yaml

kubectl apply -n postgres -f postgres-with-pv.yaml

kubectl -n postgres get pods

kubectl -n postgres exec -it postgres-0 bash

```
