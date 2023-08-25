# SonataFlow Data Index On Kubernetes

Example of how to deploy Data Index on Kubernetes.

On a terminal just run:

```shell
kubectl kustomize dataindex | kubectl apply -f -

configmap/dataindex-properties-hg9ff8bff5 created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/postgres created
```


kubectl patch svc order  -p '{"spec": {"type": "NodePort"}}'
minikube service order --url -n kogito-9741
