Case3 knative services)

    DI, with source configuration
    JS, with sink and source configuration
    The workflows with source and sink configuration

kubectl create namespace case3-kn-eventing-knservices
kubectl delete namespace case3-kn-eventing-knservices

#-2 execute this command to produce the full setup, i.e:
    * Operator deployed DI with 6 triggers
    * Operator deployed JS with 2 triggers + 1 sinkbinding
    * Operator managed workflow (including persistence) + 1 triggers , 1 sinkbindings

kubectl create namespace case3-kn-eventing-knservices

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case3-knative-services | kubectl apply -f - -n case3-kn-eventing-knservices

kn trigger list -n case3-kn-eventing-knservices

kn source list -n case3-kn-eventing-knservices


#-3 get the running pods.

kubectl get pod -n case3-kn-eventing-knservices



--------------
Useful commands for testing/verifying the use case
--------------

--------------
Curl commands
--------------
curl -X GET http://sonataflow-platform-jobs-service/q/health

curl -X GET http://sonataflow-platform-data-index-service/q/health

Urls for knative deployments:

http://callbackstatetimeouts.case3-kn-eventing-knservices

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts.case3-kn-eventing-knservices.svc.cluster.local/callbackstatetimeouts

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts.case3-kn-eventing-knservices/callbackstatetimeouts

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json'  http://eventstatetimeouts.case3-kn-eventing-knservices.svc.cluster.local/eventstatetimeouts

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json'  http://eventstatetimeouts.case3-kn-eventing-knservices/eventstatetimeouts


curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json'  http://helloworld.sonataflow-infra/helloworld

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json'  http://eventstatetimeouts.case3-kn-eventing-knservices.svc.cluster.local/eventstatetimeouts

curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json'  http://eventstatetimeouts.case3-kn-eventing-knservices/eventstatetimeouts

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



