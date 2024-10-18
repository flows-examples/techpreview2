1) create the namespace

kubectl create namespace usecase3-platform-persistence-oc

kubectl delete  namespace usecase3-platform-persistence-oc


2) create the SonataFlowPlatform

kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql | kubectl apply -f - -n usecase3-platform-persistence-oc

3) wait for DI and JS to start

kubectl get pods -n usecase3-platform-persistence-oc


-----------------------------------------
To initialize the DB manually apply these steps
-----------------------------------------

This step is no longer needed since we have configured quarkus.flyway.migrate-at-start=true

1) Create the Data base tables and the schema callbackstatetimeouts

Go to the terminal in the OpenShift webconsole for the POD of the postgres service

psql sonataflow -U sonataflow

some commands:

\l - Display database
\c - Connect to database
\dn - List schemas
\dt - List tables inside public schemas
\dt schema1. - List tables inside particular schemas. For eg: 'schema1'.

create schema callbackstatetimeouts;

set schema 'callbackstatetimeouts';

Create the tables manually:

-- To be used with kogito-addons-quarkus-persistence-jdbc for Quarkus or kogito-addons-springboot-persistence-jdbc for SpringBoot
CREATE TABLE process_instances
(
    id              character(36)     NOT NULL,
    payload         bytea             NOT NULL,
    process_id      character varying NOT NULL,
    version         bigint,
    process_version character varying,
    CONSTRAINT process_instances_pkey PRIMARY KEY (id)
);
CREATE INDEX idx_process_instances_process_id ON process_instances (process_id, id, process_version);

CREATE TABLE correlation_instances
(
    id                     character(36)         NOT NULL,
    encoded_correlation_id character varying(36) NOT NULL UNIQUE,
    correlated_id          character varying(36) NOT NULL,
    correlation            json                  NOT NULL,
    version                bigint,
    CONSTRAINT correlation_instances_pkey PRIMARY KEY (id)
);
CREATE INDEX idx_correlation_instances_encoded_id ON correlation_instances (encoded_correlation_id);
CREATE INDEX idx_correlation_instances_correlated_id ON correlation_instances (correlated_id);


2) deploy the callbackstatetimeouts workflow

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/03-sonataflow_callbackstatetimeouts-props.sw.yaml -n usecase3-platform-persistence-oc

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/04-sonataflow_callbackstatetimeouts.sw.yaml -n usecase3-platform-persistence-oc


kubectl get workflows -n usecase3-platform-persistence-oc

3) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://callbackstatetimeouts/callbackstatetimeouts



-------------------------------------
Deploy the workflowtimeouts workflow
-------------------------------------

This step is no longer needed since we have configured quarkus.flyway.migrate-at-start=true

1) Create the Data base tables and the schema workflowtimeouts

create schema wf_timeouts;

set schema 'wf_timeouts';

Create same tables as in previous WF.

2) deploy the workflowtimeouts workflow

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/05-sonataflow_workflowtimeouts.yaml -n usecase3-platform-persistence-oc

3) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://workflowtimeouts/workflowtimeouts


-----------------------------------
Deploy the helloworld timeouts
-----------------------------------

No schema initialization is required for this WF

1) deploy the helloworld workflow

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/06-sonataflow_helloworld.yaml -n usecase3-platform-persistence-oc

2) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://helloworld/helloworld


--------------
DataIndex queries
--------------
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql



