In the project root directory do:

kubectl create namespace test-platform3

kubectl kustomize platforms/dataindex_as_platform_service_ephemeral | kubectl apply -f - -n test-platform3

kubectl get pod -n test-platform3


Some initial test results:

1) The property kogito.service.url with the service address is missing, it can be added
with a property or the corresponding env var KOGITO_SERVICE_URL

other than this, looks to be working well.


