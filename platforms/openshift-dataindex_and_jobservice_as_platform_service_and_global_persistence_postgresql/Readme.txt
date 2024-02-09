1) create the namespace

    kubectl create namespace usecase3-platform-persistence-oc

2) create the SonataFlowPlatform

    kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql | kubectl apply -f - -n usecase3-platform-persistence-oc

3) wait for DI and JS to start

    kubectl get pods -m usecase3-platform-persistence-oc


-----------------------------------
Deploy the callbackstate timeouts
-----------------------------------

1) Create the Data base tables and the schema callbackstatetimeouts

Go to the terminal in the OpenShift webconsole for the POD of the postgres service

psql -U sonataflow2

some commands:

\l - Display database
\c - Connect to database
\dn - List schemas
\dt - List tables inside public schemas
\dt schema1. - List tables inside particular schemas. For eg: 'schema1'.

select current_database();

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

kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/04-sonataflow_callbackstatetimeouts.sw.yaml -n usecase3-platform-persistence-oc

kubectl get workflows -n

3) Create an instance and verify

curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'  http://callbackstatetimeouts/callbackstatetimeouts



-----------------------------------
Deploy the workflowtimeouts timeouts
-----------------------------------

1) Create the Data base tables and the schema workflowtimeouts

create schema wf_timeouts;

set schema 'wf_timeouts';

Create same tables as in previous WF


kubectl apply -f platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_postgresql/05-sonataflow_workflowtimeouts.yaml -n usecase3-platform-persistence-oc

The SonataFlow "workflowtimeouts" is invalid: spec.persistence.postgresql: Invalid value: 1: spec.persistence.postgresql in body should have at least 2 properties