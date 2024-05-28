#-1 create the namespace to support the use case

kubectl create namespace usecase3-knative-eventing

kubectl delete namespace usecase3-knative-eventing

#-2 execute this command to produce the full setup, i.e:
    * Operator managed workflow (including persistence), DI and JS deployments
    * All the necessary knative-eventing resources

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing_manual_no_sink_binding | kubectl apply -f - -n usecase3-knative-eventing


#-3 get the running pods.

kubectl get pod -n usecase3-knative-eventing



--------------
Useful commands for testing/verifying the use case
--------------

--------------
Curl commands
--------------
curl -X GET http://sonataflow-platform-jobs-service/q/health

curl -X GET http://sonataflow-platform-data-index-service/q/health

curl -X GET http://callbackstatetimeouts/q/health

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts/callbackstatetimeouts

knative -> curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts.svc.cluster.local/callbackstatetimeouts


--------------
DB queries
--------------
psql -U sonataflow

select id, process_id from process_instances;


select id, process_id from "callbackstatetimeouts".process_instances;


select id, fire_time from "jobs-service".job_details;

select * from "jobs-service".job_details;


--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql



