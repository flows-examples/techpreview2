1) create the namespace

kubectl create namespace usecase3-persistence

kubectl delete  namespace usecase3-persistence


2) create the SonataFlowPlatform

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence | kubectl apply -f - -n usecase3-persistence


3) wait for DI and JS to start

kubectl get pods -n usecase3-persistence

2) deploy the callbackstatetimeouts workflow

// kubectl apply -f platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence/04-sonataflow_callbackstatetimeouts.sw.yaml -n usecase3-persistence

kubectl kustomize usecases/usecase3-persistence | kubectl apply -f - -n usecase3-persistence

kubectl get workflows -n usecase3-persistence

3) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://callbackstatetimeouts/callbackstatetimeouts



--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql



