Case5)

    DI, JS, and the workflow takes the eventing configuration from the platform.

kubectl create namespace case5-kn-eventing
kubectl delete namespace case5-kn-eventing

kubectl create namespace case5-kn-eventing-workflows
kubectl delete namespace case5-kn-eventing-workflows

#-2 execute this command to produce the full setup, i.e:
    * Operator deployed DI with 7 triggers
    * Operator deployed JS with 2 triggers + 1 sinkbinding
    * Operator managed workflow (including persistence) + 1 trigger + 1 sinkbinding

Deploy DI and JS
kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5 | kubectl apply -f - -n case5-kn-eventing


Deploy the WF in a different namespace

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/workflows | kubectl apply -f - -n case5-kn-eventing-workflows

kubectl apply -f platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/05-configmap_callbackstatetimeouts-props.yaml -n case5-kn-eventing-workflows

kubectl apply -f platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/06-sonataflow_callbackstatetimeouts.sw.yaml -n case5-kn-eventing-workflows


kn trigger list -n case5-kn-eventing

kn source list -n case5-kn-eventing


#-3 get the running pods.

kubectl get pod -n case5-knative-eventing



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



