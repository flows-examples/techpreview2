1) create the namespace

kubectl create namespace usecase3-db-migrator

kubectl delete  namespace usecase3-db-migrator

2) create the SonataFlowPlatform

kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_dbmigrator_testing | kubectl apply -f - -n usecase3-db-migrator

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_dbmigrator_testing/02-sonataflow_platform.yaml -n usecase3-db-migrator

3) wait for DI and JS to start

kubectl get pods -n usecase3-db-migrator

4) deploy the callbackstatetimeouts workflow

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_dbmigrator_testing/03-sonataflow_callbackstatetimeouts-props.sw.yaml -n usecase3-db-migrator

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_dbmigrator_testing/04-sonataflow_callbackstatetimeouts.sw.yaml -n usecase3-db-migrator

5) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://callbackstatetimeouts/callbackstatetimeouts



--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql

