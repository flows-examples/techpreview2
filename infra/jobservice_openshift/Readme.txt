My OpenShift local is:

CRC version: 2.26.0+233df0
OpenShift version: 4.13.9

start with:

crc start -c 8 -m 19074

kubectl create namespace jobservice

kubectl kustomize infra/jobservice_openshift | kubectl apply -f - -n jobservice

kubectl get pod -n jobservice