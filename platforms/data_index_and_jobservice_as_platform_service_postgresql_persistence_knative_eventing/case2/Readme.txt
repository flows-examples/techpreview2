Case2)

    DI, with source configuration
    JS, with sink and source configuration
    The workflow with no eventing related configuration at all.
    The WF deployment will fail. (it has no way to communicate wit the DI and JS)

kubectl create namespace case2-kn-eventing
kubectl delete namespace case2-kn-eventing

#-2 execute this command to produce the full setup, i.e:
    * Operator deployed DI with 7 triggers
    * Operator deployed JS with 2 triggers + 1 sinkbinding
    * Workflow deployment must fail with the status:

    message: Error in deploy the workflow:no sink configured in the workflow or the
    platform when Job Service or Data Index Service is enabled
    reason: DeploymentFailure
    status: "False"


kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case2 | kubectl apply -f - -n case2-kn-eventing

kn trigger list -n case2-kn-eventing

kn source list -n case2-kn-eventing


#-3 get the running pods.

kubectl get pod -n case2-kn-eventing



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

select id, fire_time from "jobs-service-schema".job_details;

select id, process_id, callback_endpoint fire_time from "data-index-schema".jobs;


--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql



