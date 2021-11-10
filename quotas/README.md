CPU-Demo
========

kubectl apply -f cpu-demo.yaml

kubectl top pod cpu-demo --namespace=cpu-example

kubectl describe pod cpu-demo-2 --namespace=cpu-example

Memory-Demo
===========

kubectl apply -f memory-demo.yaml

kubectl top pod memory-demo --namespace=mem-example
