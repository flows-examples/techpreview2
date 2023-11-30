My OpenShift local is:

CRC version: 2.26.0+233df0
OpenShift version: 4.13.9

start with:

crc start -c 8 -m 19074


In the project root directory do:

kubectl create namespace test-platform3-oc

kubectl kustomize platforms/dataindex_as_platform_service_postgresql_openshift | kubectl apply -f - -n test-platform3-oc

kubectl get pod -n test-platform3-oc

Some initial test results:

1) data-index is running, properties looks good, health check pass.

2) After the platform is initialized, if we deploy the workflow the properties are ok

kubectl apply -f platforms/dataindex_as_platform_service_postgresql_openshift/04-sonataflow_callbackstatetimeouts.sw.yaml -n test-platform3-oc

Workflow passes the health check and gest the expected properties.
