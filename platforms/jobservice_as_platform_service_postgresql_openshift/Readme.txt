My OpenShift local is:

CRC version: 2.26.0+233df0
OpenShift version: 4.13.9

start with:

crc start -c 8 -m 19074


In the project root directory do:

kubectl create namespace test-platform1-oc

kubectl kustomize platforms/jobservice_as_platform_service_postgresql_openshift | kubectl apply -f - -n test-platform1-oc

kubectl get pod -n test-platform1-oc

Some initial test results:

1) jobs service has started ok, giving some time to start we can see the positive health check

curl -X GET http://sonataflow-platform-jobs-service/q/health

{
    "status": "UP",
    "checks": [
]
}


After the jobs service has started we deploy a WF:

kubectl apply -f  platforms/jobservice_as_platform_service_postgresql_openshift/04-sonataflow_callbackstatetimeouts.sw.yaml -n test-platform1-oc

2) the workflow starts ok, gets the expected properties, health check is positive.


To clean

kubectl delete namespace test-platform1-oc
kubectl delete -f  platforms/jobservice_as_platform_service_postgresql_openshift/04-sonataflow_callbackstatetimeouts.sw.yaml -n test-platform1-oc
