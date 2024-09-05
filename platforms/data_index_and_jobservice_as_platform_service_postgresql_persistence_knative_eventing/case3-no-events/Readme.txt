Case3 no-events)

DI, JS and the workflows has NO eventing configuration at all, this must fail?

This is what we have:
    DJ and JS starts well, both services starts with the old direct http execution configuration.
    It means that it's still possible to deploy these services with the direct http invocation.

    Workflows however are not deployed. Both gets a deployment failure

    callbackstatetimeouts   preview   0.0.1           False   DeploymentFailure
    eventstatetimeouts      preview   0.0.1           False   DeploymentFailure


kubectl create namespace case3-no-events
kubectl delete namespace case3-no-events

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case3-no-events | kubectl apply -f - -n case3-no-events


#-3 get the running pods.

kubectl get pod -n case3-no-events


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



