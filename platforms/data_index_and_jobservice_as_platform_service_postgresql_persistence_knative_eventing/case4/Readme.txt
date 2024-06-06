Case4)

    DI, with source configuration
    JS, with sink and source configuration
    The workflow with source and sink configuration

kubectl create namespace case4-kn-eventing
kubectl delete namespace case4-kn-eventing

#-2 execute this command to produce the full setup, i.e:
    * Operator deployed DI with 7 triggers
    * Operator deployed JS with 2 triggers + 1 sinkbinding
    * Operator managed workflow (including persistence) + 2 triggers, 1 sinkbindings

    callbackstatetimeouts-callbackevent-trigger                 sonataflow-broker    service:callbackstatetimeouts                    26m   6 OK / 6     True
    callbackstatetimeouts-callbackevent2-trigger                sonataflow-broker2   service:callbackstatetimeouts                    26m   6 OK / 6     True
    sonataflow-platform-data-index-jobs-trigger                 sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-definition-trigger   sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-error-trigger        sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-node-trigger         sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-sla-trigger          sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-state-trigger        sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-data-index-process-variable-trigger     sonataflow-broker    service:sonataflow-platform-data-index-service   29m   6 OK / 6     True
    sonataflow-platform-jobs-service-create-job-trigger         sonataflow-broker    service:sonataflow-platform-jobs-service         29m   6 OK / 6     True
    sonataflow-platform-jobs-service-delete-job-trigger         sonataflow-broker    service:sonataflow-platform-jobs-service         29m   6 OK / 6     True



kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case4 | kubectl apply -f - -n case4-kn-eventing


kn trigger list -n case4-kn-eventing

kn source list -n case4-kn-eventing


#-3 get the running pods.

kubectl get pod -n case4-kn-eventing



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



