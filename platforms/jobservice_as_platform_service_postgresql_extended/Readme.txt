In the project root directory do:

kubectl create namespace test-platform5

kubectl kustomize platforms/jobservice_as_platform_service_postgresql_extended | kubectl apply -f - -n test-platform5

kubectl get pod -n test-platform5


Some initial test results:

Init containers are working well!!!

Same comments as the standard postgresql case

1) The property kogito.service.url with the service address is missing, it can be added
with a property or the corresponding env var KOGITO_SERVICE_URL

2) the reactive datasource is not being generated

4) this env var don't apply for the jobs service QUARKUS_HIBERNATE_ORM_DATABASE_GENERATION

5) double check this: when the data-index is present, the following property must be added to the jobs service in order to
let it send the job status change events to the data-index.

// url of the data-index service
mp.messaging.outgoing.kogito-job-service-job-status-events-http.url=http://data-index-service-postgresql/jobs



