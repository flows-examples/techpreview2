In the project root directory do:

kubectl create namespace test-platform2

kubectl kustomize platforms/jobservice_as_platform_service_ephemeral | kubectl apply -f - -n test-platform2

kubectl get pod -n test-platform2


Some initial test results:

1) The property kogito.service.url with the service address is missing, it can be added
with a property or the corresponding env var KOGITO_SERVICE_URL

2) double check this: when the data-index is present, the following property must be added to the jobs service in order to
let it send the job status change events to the data-index.

// address of the data-index service
mp.messaging.outgoing.kogito-job-service-job-status-events-http.url=http://data-index-service-postgresql/jobs



