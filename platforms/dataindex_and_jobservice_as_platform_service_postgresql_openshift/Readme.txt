My OpenShift local is:

CRC version: 2.26.0+233df0
OpenShift version: 4.13.9

start with:

crc start -c 8 -m 19074


In the project root directory do:

kubectl create namespace test-platform2-oc

kubectl kustomize platforms/dataindex_and_jobservice_as_platform_service_postgresql_openshift | kubectl apply -f - -n test-platform2-oc

kubectl get pod -n test-platform2-oc

Some initial test results:

1) I can see data-index and jobs-service running, health check on both services pass!, properties looks ok.

2) After the platform is initialized, if we deploy the workflow the properties are ok

To install the workflow do:

kubectl apply -f platforms/dataindex_and_jobservice_as_platform_service_postgresql_openshift/04-sonataflow_callbackstatetimeouts.sw.yaml -n test-platform2-oc

3) I can see the workfkow installed with the expected properties for this scenario.

For Jordi's e2e tests (maybe in separate PR) I suggest by now to execute the health check for the different services.

Kind of this:

Execute a get request like this:

curl -X GET http://sonataflow-platform-jobs-service/q/health

{
    "status": "UP",
    "checks": [
        MORE IRRELEVANT STUFF HERE INSIDE
    ]
}


curl -X GET http://sonataflow-platform-data-index-service/q/health

{
    "status": "UP",
    "checks": [
        MORE IRRELEVANT STUFF HERE INSIDE
    ]
}

curl -X GET http://callbackstatetimeouts/q/health

{
    "status": "UP",
    "checks": [
        MORE IRRELEVANT STUFF HERE INSIDE
    ]
}
