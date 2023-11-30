# SonataFlow Use Cases TP2

Collection of artifacts to test SonataFlow Use Cases TP2.

## Prereqs for all the minikube use cases

1. Minikube installed

We recommend that you start Minikube with the following parameters, note that the `registry` addon must be enabled. 

```shell
minikube start --cpus 4 --memory 10240 --addons registry --addons metrics-server --insecure-registry "10.0.0.0/24" --insecure-registry "localhost:5000"
```

To verify that the registry addon was property added you can execute this command:

```shell
minikube addons list | grep registry
```

```
| registry                    | minikube | enabled ✅   | Google                         |
| registry-aliases            | minikube | disabled     | 3rd party (unknown)            |
| registry-creds              | minikube | disabled     | 3rd party (UPMC Enterprises)   |
```

2. kubectl installed

3. SonataFlow operator installed if workflows are deployed

To install the operator you can see [SonataFlow Installation](https://sonataflow.org/serverlessworkflow/latest/cloud/operator/install-serverless-operator.html).

## Use cases (Minikube) 

This is the list of available use cases for minikube:

| Use case                                                | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Deploy Data Index Locally](#deploy-data-index-locally) | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb<br/>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [Use case 1](#use-case-1)                               | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb<br/> *  The `greeting` workflow (no persistence) configured to register the process events on the Data Index Service.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [Use case 2](#use-case-2)                               | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb<br/> *  The `greeting` workflow (no persistence) <br/> * The `helloworkflow` (no persistence)<br/> * Workflows are configured to register the process events on the Data Index Service.                                                                                                                                                                                                                                                                                                                                                                                                 |
| [Use case 3](#use-case-3)                               | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb</br> * Jobs Service + postgresdb, configured to send the job events to the Data Index Service.<br/> * The `callbackstatetimeouts` (no persistence) configured to: <br/> &nbsp;&nbsp;&nbsp; - register process events on the Data Index Service<br/> &nbsp;&nbsp;&nbsp; - create timers in the Jobs Service                                                                                                                                                                                                                                                                              |
| [Use case 3 persistence](#use-case-3-persistence)       | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb</br> * Jobs Service + postgresdb, configured to send the job events to the Data Index Service.<br/> * The `callbackstatetimeouts` (with persistence) configured to: <br/> &nbsp;&nbsp;&nbsp; - register process events on the Data Index Service<br/> &nbsp;&nbsp;&nbsp; - create timers in the Jobs Service  <br/> * The sonataflow database tables are created by the user, i.e., no automatic DB initialization. This strategy is commonly used in production environment. <br/> * Shows how to use the SonataFlow CRD `spec.podTemplate` to define initContainers for the workflow. |
| [Use case 4](#use-case-4)                               | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service + postgresdb</br> * Jobs Service + postgresdb, configured to send the job events to the Data Index Service.<br/> * The `greetings` workflow (no persistence)<br/>  * The `callbackstatetimeouts` workflow (no persistence)<br/>  * The `workflowtimeouts` workflow (no persistence)<br/> * Workflows are configured to register process events on the Data Index Service and create timers on the Jobs Service                                                                                                                                                                                       |

> **NOTE:** To facilitate the switch between use cases, it's strongly recommended to install each use case in a dedicated namespace.

## Use cases (OpenShift)

This is the list of available use cases for OpenShift:

**NOTE:** Be sure your OpenShift instance has the SonataFlow Operator properly installed.

| Use case                                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Use case 3 OpenShift](#use-case-3-openshift) | This use case deploys: <br/> * PostgreSQL Service<br/> * Data Index Service (operator managed deployment) + postgresdb</br> * Jobs Service (operator managed deployment) + postgresdb, configured to send the job events to the Data Index Service.<br/> * The `callbackstatetimeouts` (no persistence) configured to: <br/> &nbsp;&nbsp;&nbsp; - register process events on the Data Index Service<br/> &nbsp;&nbsp;&nbsp; - create timers in the Jobs Service |


## Deploy Data Index locally

Example of how to deploy Data Index on Kubernetes that uses a Postgresql DB.

> **NOTE:** The workflow related use cases that needs a data index service already includes this step.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace data-index-usecase
```

2. Deploy the Data Index Service:

```shell
kubectl kustomize infra/dataindex | kubectl apply -f - -n data-index-usecase
```

```
configmap/dataindex-properties-hg9ff8bff5 created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/postgres created
```

This will deploy a Data Index for you in the `data-index-usecase` namespace. (If you don't use a namespace the `default` is used instead)
Data Index will be backed by a Postgres Data Base deployment. **This setup is not intended for production environments** since this simple Postgres Deployment does not scale well. Please see the [Postgres Operator](https://github.com/zalando/postgres-operator) for more information.


To check that the data index is running you can execute this command.

```shell
kubectl get pod -n data-index-usecase
```

```
data-index-service-postgresql-5d76dc4468-69hm6   1/1     Running   0              2m11s
postgres-7f78499688-j6282                        1/1     Running   0              2m11s
```

To access the Data Index, using Minikube you can run:

```shell
minikube service data-index-service-postgresql --url -n data-index-usecase 
```

Example output:
```
http://192.168.49.2:30352
```
The output is the Data Index URL, so you can access the GraphiQL UI by using a url like this http://192.168.49.2:30352/grpahiql/  (host and por might be different in your installation.)

For more information about Data Index and this deployment see [Data Index standalone service](https://sonataflow.org/serverlessworkflow/latest/data-index/data-index-service.html) in SonataFlow guides.

To execute queries see: [Querying Index Queries](#querying-data-index)

3. Clean the use case:

```shell
kubectl delete namespace data-index-usecase
```

## Use case 1

This use case is intended to represent an installation with:

* A singleton Data Index Service with PostgreSQL persistence
* The `greeting` workflow (no persistence), that is configured to register events to the Data Index Service. 

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase1
```

2. Deploy the Data Index Service:
```shell
kubectl kustomize infra/dataindex | kubectl apply -f - -n usecase1
```

```
configmap/dataindex-properties-hg9ff8bff5 created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/postgres created

```

Give some time for the data index to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase1
```

```
NAME                                             READY   STATUS    RESTARTS       AGE
data-index-service-postgresql-5d76dc4468-lb259   1/1     Running   0              2m11s
postgres-7f78499688-lc8n6                        1/1     Running   0              2m11s
```

3. Deploy the workflow:

```shell
 kubectl kustomize usecases/usecase1 | kubectl apply -f - -n usecase1
 ```

```
configmap/greeting-props created
sonataflow.sonataflow.org/greeting created
sonataflowplatform.sonataflow.org/sonataflow-platform created
```

Give some time for the sonataflow operator to build and deploy the workflow. 
To check that the workflow is ready you can use this command.

```shell
kubectl get workflow -n usecase1
```

```
NAME       PROFILE   VERSION   URL   READY   REASON
greeting             0.0.1           True    
```

4. Expose the workflow and get the url:

```shell
kubectl patch svc greeting -p '{"spec": {"type": "NodePort"}}' -n usecase1
```

```shell
 minikube service greeting --url -n usecase1
 ```

5. Create a workflow instance:

You must use the URLs calculated in step 4.

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{"name": "John", "language": "English"}'    http://192.168.49.2:32407/greeting
```

**To execute queries and see the workflows information see:** [Querying Index Queries](#querying-data-index)


6. Clean the use case:

```shell
kubectl delete namespace usecase1
```

## Use case 2

This use case is intended to represent an installation with:

* A singleton Data Index Service with PostgreSQL persistence
* The `greeting` workflow (no persistence)
* The `helloworkflow` workflow (no persistence)
* The workflows are configured to register the process events on the Data Index Service.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase2
```

2. Deploy the Data Index Service:
```shell
kubectl kustomize infra/dataindex | kubectl apply -f - -n usecase2
```

```
configmap/dataindex-properties-hg9ff8bff5 created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/postgres created

```

Give some time for the data index to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase2
```

```
NAME                                             READY   STATUS    RESTARTS       AGE
data-index-service-postgresql-5d76dc4468-lb259   1/1     Running   0              2m11s
postgres-7f78499688-lc8n6                        1/1     Running   0              2m11s
```

3. Deploy the workflows:

```shell
 kubectl kustomize usecases/usecase2 | kubectl apply -f - -n usecase2
 ```

```
configmap/greeting-props created
configmap/helloworld-props created
sonataflow.sonataflow.org/greeting created
sonataflow.sonataflow.org/helloworld created
sonataflowplatform.sonataflow.org/sonataflow-platform created
```

Give some time for the sonataflow operator to build and deploy the workflows.
To check that the workflows are ready you can use this command.

```shell
kubectl get workflow -n usecase2
```

```
NAME       PROFILE   VERSION   URL   READY   REASON
greeting             0.0.1           True
helloworld           0.0.1           True        
```

4. Expose the workflows and get the urls:

```shell
kubectl patch svc greeting helloworld  -p '{"spec": {"type": "NodePort"}}' -n usecase2
```

```shell
 minikube service greeting --url -n usecase2
 ```

```shell
 minikube service helloworld --url -n usecase2
 ```

5. Create a workflow instances:

You must use the URLs calculated in step 4.

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{"name": "John", "language": "English"}'    http://192.168.49.2:32407/greeting
```

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'    http://192.168.49.2:32327/helloworld
```

**To execute queries and see the workflows information see:** [Querying Index Queries](#querying-data-index)

6. Clean the use case:

```shell
kubectl delete namespace usecase2
```

## Use case 3

This use case is intended to represent an installation with:

* A singleton Data Index Service with PostgreSQL persistence
* A singleton Jobs Service with PostgreSQL persistence configured to send job status events to the Data Index Service.
* The `callbackstateworkflow` workflow (no persistence)
* The workflow is configured to register the process events on the Data Index Service.
* The workflow is configured to create timers on the Jobs Service.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase3
```

2. Install the Data Index Service and the Jobs Service:

```shell
kubectl kustomize infra/dataindex_and_jobservice | kubectl apply -f - -n usecase3
```

```
configmap/dataindex-properties-hg9ff8bff5 created
configmap/jobs-service-properties-9bf9cg9b9f created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/jobs-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/jobs-service-postgresql created
deployment.apps/postgres created
```

Give some time for the data index and the job service to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase3
```

```
NAME                                             READY   STATUS    RESTARTS   AGE
data-index-service-postgresql-69f684d458-scxbr   1/1     Running   0          65s
jobs-service-postgresql-5c9b74cfc5-qnvkh         1/1     Running   0          65s
postgres-7f78499688-2dnj5                        1/1     Running   0          65s
```

3. Install the workflow:

```shell
 kubectl kustomize usecases/usecase3 | kubectl apply -f - -n usecase3
 ```

```
configmap/callbackstatetimeouts-props created
sonataflow.sonataflow.org/callbackstatetimeouts created
sonataflowplatform.sonataflow.org/sonataflow-platform created
```

Give some time for the sonataflow operator to build and deploy the workflow.
To check that the workflow is ready you can use this command.

```shell
kubectl get workflow -n usecase3
```

```
NAME                    PROFILE   VERSION   URL   READY   REASON
callbackstatetimeouts             0.0.1           True      
```

4. Expose the workflow and get the url: 

```shell
kubectl patch svc callbackstatetimeouts  -p '{"spec": {"type": "NodePort"}}' -n usecase3
```

```shell
 minikube service callbackstatetimeouts --url -n usecase3
 ```

5. Create a workflow instance:

You must use the URL calculated in step 4.

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'     http://192.168.49.2:30707/callbackstatetimeouts
```

**To execute queries and see the workflows information see:** [Querying Index Queries](#querying-data-index)

6. Clean the use case:

```shell
kubectl delete namespace usecase3
```

## Use case 3 persistence

This use case is intended to represent an installation with:

* A singleton Data Index Service with PostgreSQL persistence
* A singleton Jobs Service with PostgreSQL persistence configured to send job status events to the Data Index Service.
* The `callbackstateworkflow` workflow (with persistence)
* The sonataflow database tables are created by the user, i.e., no automatic DB initialization. This strategy is commonly used in production environment.
* The workflow is configured to register the process events on the Data Index Service.
* The workflow is configured to create timers on the Jobs Service.
* Shows how to use the SonataFlow CRD `spec.podTemplate` to define initContainers for the workflow.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase3-persistence
```

2. Install the Data Index Service and the Jobs Service:

```shell
kubectl kustomize infra/dataindex_and_jobservice | kubectl apply -f - -n usecase3-persistence
```

```
configmap/dataindex-properties-hg9ff8bff5 created
configmap/jobs-service-properties-9bf9cg9b9f created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/jobs-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/jobs-service-postgresql created
deployment.apps/postgres created
```

Give some time for the data index and the job service to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase3-persistence
```

```
NAME                                             READY   STATUS    RESTARTS      AGE
data-index-service-postgresql-69f684d458-scxbr   1/1     Running   0          65s
jobs-service-postgresql-5c9b74cfc5-qnvkh         1/1     Running   0          65s
postgres-7f78499688-2dnj5                        1/1     Running   0          65s
```

3. Visit [this link](https://repo1.maven.org/maven2/org/kie/kogito/kogito-ddl) and get the sonataflow DDL scripts to create the tables. 
You must download the scripts according to the sonataflow version that you are using, for example, [../org/kie/kogito/kogito-ddl/1.44.1.Final/kogito-ddl-1.44.1.Final-db-scripts.zip](https://repo1.maven.org/maven2/org/kie/kogito/kogito-ddl/https://repo1.maven.org/maven2/org/kie/kogito/kogito-ddl/1.44.1.Final/kogito-ddl-1.44.1.Final-db-scripts.zip)
and extract the xx.xx.x__create_runtime_PostgreSQL.sql file for postgresql, for example `postgresql/V1.35.0__create_runtime_PostgreSQL.sql`.

4. Execute this command to expose the `postgres` service created in step 1, and get it's ip address and port. 

```shell
kubectl patch svc postgres -p '{"spec": {"type": "NodePort"}}' -n usecase3-persistence
```

To get the ip address and port of the exposed service you must execute this command:

```shell
minikube service postgres  --url -n usecase3-persistence
```

With the ip address and port calculated above, you must execute this command, and when you are asked for a password you must use `sonataflow`:

```
psql -H -h 192.168.49.2 -p 32321 -U sonataflow -d sonataflow -a -f V1.35.0__create_runtime_PostgreSQL.sql
```

After executing the command you will see an output similar to this:
```shell
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
CREATE TABLE
CREATE INDEX idx_process_instances_process_id ON process_instances (process_id, id, process_version);
CREATE INDEX
CREATE TABLE correlation_instances
(
    id                     character(36)         NOT NULL,
    encoded_correlation_id character varying(36) NOT NULL UNIQUE,
    correlated_id          character varying(36) NOT NULL,
    correlation            json                  NOT NULL,
    version                bigint,
    CONSTRAINT correlation_instances_pkey PRIMARY KEY (id)
);
CREATE TABLE
CREATE INDEX idx_correlation_instances_encoded_id ON correlation_instances (encoded_correlation_id);
CREATE INDEX
CREATE INDEX idx_correlation_instances_correlated_id ON correlation_instances (correlated_id);
CREATE INDEX
```

> **NOTE:** The DB service exposure and the database tables initialization is an example procedure. In production environments, even when the selected DDL is the same, you might probably apply another procedure to access the DB, etc.


5. Install the workflow:

```shell
 kubectl kustomize usecases/usecase3-persistence | kubectl apply -f - -n usecase3-persistence
 ```

```
configmap/callbackstatetimeouts-props created
sonataflow.sonataflow.org/callbackstatetimeouts created
sonataflowplatform.sonataflow.org/sonataflow-platform created
```

Give some time for the sonataflow operator to build and deploy the workflow.
To check that the workflow is ready you can use this command.

```shell
kubectl get workflow -n usecase3-persistence
```

```
NAME                    PROFILE   VERSION   URL   READY   REASON
callbackstatetimeouts             0.0.1           True      
```

6. Expose the workflow and get the url:

```shell
kubectl patch svc callbackstatetimeouts  -p '{"spec": {"type": "NodePort"}}' -n usecase3-persistence
```

```shell
 minikube service callbackstatetimeouts --url -n usecase3-persistence
 ```

7. Create a workflow instance:

You must use the URL calculated in step 6.

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'     http://192.168.49.2:30707/callbackstatetimeouts
```

**To execute queries and see the workflows information see:** [Querying Index Queries](#querying-data-index)

8. Clean the use case:

```shell
kubectl delete namespace usecase3-persistence
```

## Use case 4

This use case is intended to represent installation with:

* A singleton Data Index Service with PostgreSQL persistence
* A singleton Jobs Service with PostgreSQL persistence configured to send job status events to the Data Index Service.
* The `greeting` workflow (no persistence)
* The `callbackstateworkflow` workflow (no persistence)
* The `workflowtimeouts` workflow (no persistence)
* The workflows are configured to register the process events on the Data Index Service.
* The workflows are configured to create timers on the Jobs Service.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase4
```

2. Install the Data Index Service and the Jobs Service:

```shell
kubectl kustomize infra/dataindex_and_jobservice | kubectl apply -f - -n usecase4
```

```
configmap/dataindex-properties-hg9ff8bff5 created
configmap/jobs-service-properties-9bf9cg9b9f created
secret/postgres-secrets-22tkgc2dt7 created
service/data-index-service-postgresql created
service/jobs-service-postgresql created
service/postgres created
persistentvolumeclaim/postgres-pvc created
deployment.apps/data-index-service-postgresql created
deployment.apps/jobs-service-postgresql created
deployment.apps/postgres created
```

Give some time for the data index and the job service to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase4
```

```
NAME                                             READY   STATUS    RESTARTS      AGE
data-index-service-postgresql-69f684d458-scxbr   1/1     Running   0          65s
jobs-service-postgresql-5c9b74cfc5-qnvkh         1/1     Running   0          65s
postgres-7f78499688-2dnj5                        1/1     Running   0          65s
```

3. Install the workflows:

```shell
 kubectl kustomize usecases/usecase4 | kubectl apply -f - -n usecase4
 ```

```
configmap/callbackstatetimeouts-props created
configmap/greeting-props created
configmap/workflowtimeouts-props created
sonataflow.sonataflow.org/callbackstatetimeouts created
sonataflow.sonataflow.org/greeting created
sonataflow.sonataflow.org/workflowtimeouts created
sonataflowplatform.sonataflow.org/sonataflow-platform created
```

Give some time for the sonataflow operator to build and deploy the workflow.
To check that the workflows are ready you can use this command.

```shell
kubectl get workflow -n usecase4
```

```
NAME                    PROFILE   VERSION   URL   READY   REASON
callbackstatetimeouts             0.0.1           True    
greeting                          0.0.1           True    
workflowtimeouts                  0.0.1           True   
```

4. Expose the workflows and get the urls:

```shell
kubectl patch svc greeting callbackstatetimeouts workflowtimeouts -p '{"spec": {"type": "NodePort"}}' -n usecase4
```

```shell
minikube service greeting --url -n usecase4
 ```

```shell
minikube service callbackstatetimeouts --url -n usecase4
 ```

```shell
minikube service workflowtimeouts --url -n usecase4
```

5. Create a workflow instances:

You must use the URLs calculated in step 4.

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{"name": "John", "language": "English"}'    http://192.168.49.2:32407/greeting
```

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'     http://192.168.49.2:30707/callbackstatetimeouts
```

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'     http://192.168.49.2:30083/workflowtimeouts
```

**To execute queries and see the workflows information see:** [Querying Index Queries](#querying-data-index)


6. Clean the use case:

```shell
kubectl delete namespace usecase4
```


## Use case 3 OpenShift

This use case is intended to represent an installation with:

* A singleton Data Index Service (SonataFlow operator based deployment) with PostgreSQL persistence
* A singleton Jobs Service(SonataFlow operator based deployment) with PostgreSQL persistence configured to send job status events to the Data Index Service.
* The `callbackstateworkflow` workflow (no persistence)
* The workflow is configured to register the process events on the Data Index Service.
* The workflow is configured to create timers on the Jobs Service.

### Procedure

Open a terminal and run the following commands:

1. Create the namespace:

```shell
kubectl create namespace usecase3-oc
```

2. Install the Data Index Service and the Jobs Service by using a Sonataflow Plaform

```shell
kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_postgresql | kubectl apply -f - -n usecase3-oc
```

```
persistentvolumeclaim/postgres-pvc created
deployment.apps/postgres created
service/postgres created
sonataflowplatform.sonataflow.org/sonataflow-platform created
secret/postgres-secrets created
```

Give some time for the data index and the job service to start, you can check that it's running by executing.

```shell
kubectl get pod -n usecase3-oc
```

```
NAME                                                      READY   STATUS    RESTARTS   AGE
postgres-684bc5876b-69phn                                 1/1     Running   0          100s
sonataflow-platform-data-index-service-6b6878d6cc-t5fzn   1/1     Running   0          100s
sonataflow-platform-jobs-service-5d4f89cd9d-kcn54         1/1     Running   0          100s
```

3. Install the workflow:

```shell
 kubectl kustomize usecases/usecase3-oc | kubectl apply -f - -n usecase3-oc
 ```

```
sonataflow.sonataflow.org/callbackstatetimeouts created
```

Give some time for the sonataflow operator to build and deploy the workflow.
To check that the workflow is ready you can use this command.

```shell
kubectl get workflow -n usecase3-oc
```

```
NAME                    PROFILE   VERSION   URL   READY   REASON
callbackstatetimeouts             0.0.1           True      
```

4. Expose the workflow and get the url:

TODO (create a route)

5. Create a workflow instance:

TODO (use the route)

Using the OpenShift web console, go to the terminal of any of the pods in the namespace usecase3-oc and do.
(NOTE: to create the first instance might take some time depending on your OpenShift instance specially if you are using OpenShift local)

```shell
curl -X POST -H 'Content-Type:application/json' -H 'Accept:application/json' -d '{}'     http://callbackstatetimeouts/callbackstatetimeouts
```

You'll see an output like this: {"id":"6aa23153-2dbf-4da4-b4e0-95a87e582e46","workflowdata":{}}

To query the data-index and see the information for the just created WF instance, do:

```shell
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service.test-platform2-oc/graphql
```

You'll see an output like this: {"data":{"ProcessInstances":[{"id":"3c54f56c-1229-40b7-b27f-925f57bc73d8","processId":"callbackstatetimeouts","state":"COMPLETED","endpoint":"http://callbackstatetimeouts.test-platform2-oc/callbackstatetimeouts","serviceUrl":"http://callbackstatetimeouts.test-platform2-oc","start":"2024-01-11T15:49:26.688Z","end":null}]}}

To query the data-index and see the information for the just created Job instance, do:

```shell
curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service.test-platform2-oc/graphql
```

6. Clean the use case:

```shell
kubectl delete namespace usecase3-oc
```


## Querying Data Index

You can use the public Data Index endpoint to play around with the GraphiQL interface.

### Procedure

This procedure apply to all use cases with that deploys the Data Index Service.

1. Get the Data Index Url:

```shell
minikube service data-index-service-postgresql --url -n my_usecase
```

2. Open the GrahiqlUI

Using the url returned in 1, open a browser window in the following url http://192.168.49.2:32409/graphiql/, note that IP and port will be different in your installation, and don't forget to add the last slash "/" to the url, otherwise the GraphiqlUI won't be opened.  


To see the process instances information you can execute this query:

```graphql
{
  ProcessInstances {
    id,
    processId,
    processName,
    variables,
    state,
    endpoint,
    serviceUrl,
    start,
    end
  }
}
```

The results should be something like:


```json
{
  "data": {
    "ProcessInstances": [
      {
        "id": "3ed8bf63-85c9-425d-9099-49bfb63608cb",
        "processId": "greeting",
        "processName": "workflow",
        "variables": "{\"workflowdata\":{\"name\":\"John\",\"greeting\":\"Hello from JSON Workflow, \",\"language\":\"English\"}}",
        "state": "COMPLETED",
        "endpoint": "/greeting",
        "serviceUrl": "http://greeting",
        "start": "2023-09-13T06:59:24.319Z",
        "end": "2023-09-13T06:59:24.400Z"
      }
    ]
  }
}
```

To see the jobs instances information, if any, you can execute this query:

```graphql
{
  Jobs {
    id,
    processId,
    processInstanceId,
    status,
    expirationTime,
    retries,
    endpoint,
    callbackEndpoint
  }
}
```

The results should be something like:

```json
{
  "data": {
    "Jobs": [
      {
        "id": "55c7aadb-3dff-4b97-af8e-cc45014b1c0d",
        "processId": "callbackstatetimeouts",
        "processInstanceId": "299886b7-2b78-4965-a701-16783c4162d8",
        "status": "EXECUTED",
        "expirationTime": null,
        "retries": 0,
        "endpoint": "http://jobs-service-postgresql/jobs",
        "callbackEndpoint": "http://callbackstatetimeouts:80/management/jobs/callbackstatetimeouts/instances/299886b7-2b78-4965-a701-16783c4162d8/timers/-1"
      }
    ]
  }
}
```
