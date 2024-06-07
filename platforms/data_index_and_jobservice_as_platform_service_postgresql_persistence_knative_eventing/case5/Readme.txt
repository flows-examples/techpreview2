Case5)

    1) DI, JS, and the workflow takes the eventing configuration from the sonataflow-platform in namespace case5-kn-eventing
    2) We create also a cluster-platform
    3) Workflows deployed in case5-kn-eventing must take the Broker configuration automatically.
    4) Workflows deployed in case5-kn-eventing-workflows must also take the Broker configuration automatically, since we created the cluster-platform.

kubectl create namespace case5-kn-eventing
kubectl delete namespace case5-kn-eventing

kubectl create namespace case5-kn-eventing-workflows
kubectl delete namespace case5-kn-eventing-workflows

Deploy DI and JS:
kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5 | kubectl apply -f - -n case5-kn-eventing


Deploy the WF in a different namespace: case5-kn-eventing-workflows

kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing/case5/workflows | kubectl apply -f - -n case5-kn-eventing-workflows

------------------
Issue
------------------

The workflows throws this exception when starting:

2024-06-07 16:20:46,581 WARN [org.kie.kog.add.qua.kna.eve.KnativeEventingConfigSourceFactory] (main) K_SINK variable is empty or doesn't exist. Please make sure that this service is a Knative Source or has a SinkBinding bound to it.
2024-06-07 16:20:47,316 ERROR [io.sma.rea.mes.provider] (main) SRMSG00230: Unable to create the publisher or subscriber during initialization: java.lang.IllegalArgumentException: The attribute `url` on connector 'quarkus-http' (channel: kogito_outgoing_stream) must be set
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnectorOutgoingConfiguration.lambda$getUrl$0(QuarkusHttpConnectorOutgoingConfiguration.java:28)
at java.base/java.util.Optional.orElseThrow(Optional.java:403)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnectorOutgoingConfiguration.getUrl(QuarkusHttpConnectorOutgoingConfiguration.java:28)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnectorOutgoingConfiguration.validate(QuarkusHttpConnectorOutgoingConfiguration.java:97)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnectorOutgoingConfiguration.<init>(QuarkusHttpConnectorOutgoingConfiguration.java:16)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnector.getSubscriberBuilder(QuarkusHttpConnector.java:98)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnector_Subclass.getSubscriberBuilder$$superforward(Unknown Source)
at io.quarkus.reactivemessaging.http.runtime.QuarkusHttpConnector_Subclass$$function$$2.apply(Unknown Source)
at io.quarkus.arc.impl.AroundInvokeInvocationContext.proceed(AroundInvokeInvocationContext.java:73)

And this is because the SinkBinding was created as expected, but with the following configuration:
Note: the trigger for the workflow is created ok.

  kind: SinkBinding
  metadata:
    annotations:
      sources.knative.dev/creator: system:serviceaccount:sonataflow-operator-system:sonataflow-operator-controller-manager
      sources.knative.dev/lastModifier: system:serviceaccount:sonataflow-operator-system:sonataflow-operator-controller-manager
    creationTimestamp: "2024-06-07T16:14:32Z"
    finalizers:
    - sinkbindings.sources.knative.dev
    generation: 1
    labels:
      app: callbackstatetimeouts
      sonataflow.org/workflow-app: callbackstatetimeouts
      sonataflow.org/workflow-namespace: case5-kn-eventing-workflows
    name: callbackstatetimeouts-sb
    namespace: case5-kn-eventing-workflows
    ownerReferences:
    - apiVersion: sonataflow.org/v1alpha08
      blockOwnerDeletion: true
      controller: true
      kind: SonataFlow
      name: callbackstatetimeouts
      uid: 1513d9e5-5475-4e42-a24d-c1599d6064fe
    resourceVersion: "251448"
    uid: 6e162da5-8c50-440d-8db7-3ea4a983e79a
  spec:
    sink:
      ref:
        apiVersion: eventing.knative.dev/v1
        kind: Broker
        name: sonataflow-broker
        namespace: case5-kn-eventing-workflows   (THE BROKER IS in case5-kn-eventing instead)
    subject:
      apiVersion: apps/v1
      kind: Deployment
      name: callbackstatetimeouts
      namespace: case5-kn-eventing-workflows






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



