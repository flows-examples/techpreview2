Steps to reproduce the Parodos infrastructure.

kubectl create namespace sonataflow-infra

kubectl kustomize  | kubectl apply -f - -n sonataflow-infra

kubectl apply -f 02-sonataflow-platform.yaml -n sonataflow-infra

kubectl get pod -n sonataflow-infra

kubectl apply -f 03-cluster-platform.yaml

(optional for the knative eventing integration)

kubectl apply -f 04-default-broker.yaml -n sonataflow-infra

kubectl create namespace test1

kubectl apply -f 05-callbackstatetimeouts-knative.yaml -n test1

To facilitate querying the data-index:

minikube addons enable ingress -p knative

kubectl apply  -f ../../infra/common/data-index-service-ingress.yaml -n sonataflow-infra

minikube ip -p knative

And opening the following url with the returned ip: http://192.168.58.2/graphiql/



Curl example commands to create a workflow instance and query the data-index.

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://callbackstatetimeouts/callbackstatetimeouts

curl -H 'Content-Type: application/json' -H 'Accept: application/json' -X POST -d '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service.sonataflow-infra/graphql

curl -H 'Content-Type: application/json' -H 'Accept: application/json' -X POST -d '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service.sonataflow-infra/graphql

curl -H 'Content-Type: application/json' -H 'Accept: application/json' -X POST -d '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service.sonataflow-infra/graphql
