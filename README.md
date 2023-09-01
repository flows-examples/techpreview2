# SonataFlow Data Index On Kubernetes

Example of how to deploy Data Index on Kubernetes.

## Deploy Data Index locally

### Prereqs

1. Minikube installed
2. kubectl installed

### Procedure

Open a terminal and run the following command:

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

This will deploy a Data Index for you in the `default` namespace. Alternatively, you can pass `-n <namespace>` to deploy it in a specific namespace.
Data Index will be backed by a Postgres Data Base deployment. **This setup is not intended for production environments** since this simple Postgres Deployment does not scale well. Please see the [Postgres Operator](https://github.com/zalando/postgres-operator) for more information.

To access the Data Index, using Minikube you can run:

```shell
minikube service data-index-service-postgresql --url 
```

The output is the Data Index URL, so you can access the GraphiQL UI.

For more information about Data Index and this deployment see [Data Index standalone service](https://sonataflow.org/serverlessworkflow/latest/data-index/data-index-service.html) in SonataFlow guides.

## Deploy a SonataFlow example

To feed Data Index with workflow execution data, you must deploy a SonataFlow instance and configure it to send events to Data Index.

### Prereqs

1. Install Data Index (see above)
2. Install the SonataFlow Operator on Minikube; see the instructions in the [operator installation guide](https://sonataflow.org/serverlessworkflow/latest/cloud/operator/install-serverless-operator.html).

### Procedure

Open a new terminal and run:

```shell
kubectl apply -f sonataflow/
```

It should build and deploy a new instance of the Greetings workflow in the same namespace where Data Index is deployed.

To access the application, run:

```shell
kubectl patch svc greeting  -p '{"spec": {"type": "NodePort"}}'
minikube service greeting --url
```

This should expose the workflow and return the public endpoint.

You can then use it to `curl` the REST endpoint and start a new workflow instance which will return immediately:

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{"name": "John", "language": "English"}' http://${ENDPOINT}/greeting
```

Replace `${ENDPOINT}` with the output from the previous command.

## Querying Data Index

You can use the public Data Index endpoint to play around with the GraphiQL interface. Try submiting:

```graphql
{
    ProcessInstances {
        id
        processName
        endpoint
        variables
        serviceUrl
    }
}
```

The results should be something like:

```json
{
   "data":{
       "ProcessInstances":[
           {
               "id":"3ed8bf63-85c9-425d-9099-49bfb63608cb",
               "processName":"workflow",
               "endpoint":"/greeting",
               "variables":"{\"workflowdata\":{\"name\":\"John\",\"greeting\":\"Hello from JSON Workflow, \",\"language\":\"English\"}}",
               "serviceUrl":null     
      }   
    ] 
  }
}
```
