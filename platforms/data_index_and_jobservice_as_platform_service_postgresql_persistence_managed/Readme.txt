kubectl create namespace usecase3-persistence

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_managed | kubectl apply -f - -n usecase3-persistence

kubectl get pod -n usecase3-persistence

kubectl kustomize usecases/usecase3-persistence-managed | kubectl apply -f - -n usecase3-persistence

kubectl delete namespace usecase3-persistence




--------------
Curl commands
--------------
curl -H 'Content-Type: application/json' -H 'Accept: application/json' -X POST http://callbackstatetimeouts/callbackstatetimeouts

curl -H 'Content-Type: application/json' -H 'Accept: application/json' -X POST http://callbackstatetimeouts.svc.cluster.local/callbackstatetimeouts


--------------
DB queries
--------------
psql -U sonataflow

select id, process_id from process_instances;


select id, process_id from "callbackstatetimeouts".process_instances;


select id, fire_time from "jobs-service".job_details;

select * from "jobs-service".job_details;


curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql


curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql


