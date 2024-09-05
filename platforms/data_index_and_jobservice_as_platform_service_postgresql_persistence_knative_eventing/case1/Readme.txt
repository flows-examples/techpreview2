Case1)
    Broker is configured at the platform level: spec.eventing.broker

    DI, JS, has no specific eventing configuration, and thus, just take
    the eventing configuration from the platform.

    Workflow has no sink or sources configuration, and thus, just take
    the eventing configuration from the platform.

kubectl create namespace case1-kn-eventing
kubectl delete namespace case1-kn-eventing

#-2 execute this command to produce the full setup, i.e:
    * Operator deployed DI with 7 triggers
    * Operator deployed JS with 2 triggers + 1 sinkbinding
    * Operator managed workflow (including persistence) + 1 trigger + 1 sinkbinding

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case1 | kubectl apply -f - -n case1-kn-eventing


kn trigger list -n case1-kn-eventing

kn source list -n case1-kn-eventing


#-3 get the running pods.

kubectl get pod -n usecase3-knative-eventing


kogito.jobs-service.management.leader-check.expiration-in-seconds

kogito_jobs_service_management_leader_check_expiration_in_seconds

KOGITO_JOBS_SERVICE_MANAGEMENT_LEADER_CHECK_EXPIRATION_IN_SECONDS
--------------
Useful commands for testing/verifying the use case
--------------

--------------
Curl commands
--------------

curl -X GET http://localhost:8080/q/health/live

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

select id, fire_time from "jobs-service-schema".job_details;

select id, process_id, callback_endpoint fire_time from "data-index-schema".jobs;


--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql



