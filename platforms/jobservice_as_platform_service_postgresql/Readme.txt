In the project root directory do:

kubectl create namespace test-platform1

kubectl kustomize platforms/jobservice_as_platform_service_postgresql | kubectl apply -f - -n test-platform1

kubectl get pod -n test-platform1


Some initial test results:

1) The property kogito.service.url with the service address is missing, it can be added
with a property or the corresponding env var KOGITO_SERVICE_URL

2) the reactive datasource is not being generated

3) double check this: when the data-index is present, the following property must be added to the jobs service in order to
let it send the job status change events to the data-index.

// url of the data-index service
mp.messaging.outgoing.kogito-job-service-job-status-events-http.url=http://data-index-service-postgresql/jobs



In a later point in time we try to deploy the following workflow:

kubectl apply -f  platforms/jobservice_as_platform_service_postgresql/04-sonataflow_callbackstatetimeouts.sw.yaml -n test-platform1
