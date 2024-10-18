Important!, Prometheus operator must be installed in namespace usecase3-platform-persistence-monitoring-oc

The use case creates,
    * A Prometheus instance
    * A postgresql server instance
    * The SFP with spec.monitoring.enabled: true + persistence
    * The callbackstatetimeouts

1) create the namespace

kubectl create namespace usecase3-platform-persistence-monitoring-oc

kubectl delete  namespace usecase3-platform-persistence-monitoring-oc

2) Important, install the Prometheus operator in the namespace usecase3-platform-persistence-monitoring-oc using the OpenShift UI

3) create the Prometheus instance

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_monitoring/00-sonataflow_prometheus.yaml -n usecase3-platform-persistence-monitoring-oc

4) create the SonataFlowPlatform

kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_monitoring | kubectl apply -f - -n usecase3-platform-persistence-monitoring-oc

5) wait for DI and JS to start

kubectl get pods -n usecase3-platform-persistence-monitoring-oc

6) deploy the callbackstatetimeouts workflow

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_monitoring/03-sonataflow_callbackstatetimeouts-props.sw.yaml -n usecase3-platform-persistence-monitoring-oc

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql_monitoring/04-sonataflow_callbackstatetimeouts.sw.yaml -n usecase3-platform-persistence-monitoring-oc

kubectl get workflows -n usecase3-platform-persistence-monitoring-oc

When the workflow pod is up and running we must see the corresponding ServiceMonitor
(note, we can see the servicemonitor is created even before the pod is ready 1/1)

kubectl get servicemonitor -n usecase3-platform-persistence-monitoring-oc

7) Create an instance and verify that we have metrics in in Prometheus:

For example in the web console terminal for the callbackstatetimeouts POD, execute:

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{"name":"bob","surname":"esponja"}'  http://callbackstatetimeouts/callbackstatetimeouts

curl http://prometheus-operated.usecase3-platform-persistence-monitoring-oc:9090/api/v1/query --data-urlencode 'query=kogito_process_instance_duration_seconds_count{job="callbackstatetimeouts",namespace="usecase3-platform-persistence-monitoring-oc"}'

curl http://prometheus-operated.usecase3-platform-persistence-monitoring-oc:9090/api/v1/query --data-urlencode 'query=kogito_process_instance_started_total{job="callbackstatetimeouts",namespace="usecase3-platform-persistence-monitoring-oc"}'

We must see the respective metrics values.

--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql


