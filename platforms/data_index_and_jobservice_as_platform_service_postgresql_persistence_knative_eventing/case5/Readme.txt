Case5)

    1) DI, JS, and the workflow takes the eventing configuration from the sonataflow-platform in namespace case5-kn-eventing
    2) We create also a cluster-platform
    3) Workflows deployed in case5-kn-eventing must take the Broker configuration automatically.
    4) Workflows deployed in case5-kn-eventing-workflows must also take the Broker configuration automatically, since we created the cluster-platform.

kubectl create namespace case5-kn-eventing
kubectl delete namespace case5-kn-eventing

kubectl create namespace case5-kn-eventing-workflows
kubectl delete namespace case5-kn-eventing-workflows

Step 1) Deploy DI, JS and the cluster platform.

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5 | kubectl apply -f - -n case5-kn-eventing


Step 2) Deploy the callbackstatetimeouts WF in a different namespace: case5-kn-eventing-workflows
In this case, the WF takes the eventing configurations from the cluster platform.

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/workflows | kubectl apply -f - -n case5-kn-eventing-workflows


kn trigger list -n case5-kn-eventing

kn source list -n case5-kn-eventing


#-3 get the running pods.

kubectl get pod -n case5-knative-eventing


Step 3) Deploy the callbackstatetimeouts2 WF in the namespace: case5-kn-eventing:
In this case the WF takes the eventing configurations from the current platform.

kubectl apply -f platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/workflows/07-configmap_callbackstatetimeouts2-props.yaml -n case5-kn-eventing

kubectl apply -f platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/workflows/08-sonataflow_callbackstatetimeouts2.sw.yaml -n case5-kn-eventing


After executing Step1, Step2 and Step3 all workflows must ok.




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



