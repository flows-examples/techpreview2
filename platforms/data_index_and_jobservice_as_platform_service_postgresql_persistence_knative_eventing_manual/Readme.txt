kubectl create namespace usecase3-knative-eventing

kubectl delete namespace usecase3-knative-eventing


kubectl kustomize platforms/data_index_and_jobservice_as_platform_service_postgresql_persistence_knative_eventing_manual | kubectl apply -f - -n usecase3-knative-eventing

kubectl get pod -n usecase3-knative-eventing

curl -X GET http://sonataflow-platform-jobs-service/q/health

curl -X GET http://sonataflow-platform-data-index-service/q/health

curl -X GET http://callbackstatetimeouts/q/health

--------------
Curl commands
--------------
curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts/callbackstatetimeouts

knative -> curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' http://callbackstatetimeouts.svc.cluster.local/callbackstatetimeouts


--------------
DB queries
--------------
psql -U sonataflow

select id, process_id from process_instances;


select id, process_id from "callbackstatetimeouts".process_instances;


select id, fire_time from "jobs-service".job_details;

select * from "jobs-service".job_details;


curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessInstances { id, processId, state, endpoint, serviceUrl, start, end } }"  }' http://sonataflow-platform-data-index-service/graphql

curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ ProcessDefinitions { id, serviceUrl, source } }"  }' http://sonataflow-platform-data-index-service/graphql


curl -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data '{"query" : "{ Jobs { id, processId, processInstanceId, status, expirationTime, retries, endpoint, callbackEndpoint } }"  }' http://sonataflow-platform-data-index-service/graphql














quarkus_smallrye_health_check__org_kie_kogito_jobs_service_messaging_http_health_knative_KSinkInjectionHealthCheck__enabled

QUARKUS_SMALLRYE_HEALTH_CHECK__IO_QUARKUS_KAFKA_CLIENT_HEALTH_KAFKAHEALTHCHECK__ENABLED

QUARKUS_SMALLRYE_HEALTH_CHECK__ORG_KIE_KOGITO_JOBS_SERVICE_MESSAGING_HTTP_HEALTH_KNATIVE_KSINKINJECTIONHEALTHCHECK__ENABLED



mp_messaging_outgoing_kogito_job_service_job_request_events_url = http://sonataflow-platform-jobs-service.usecase3-persistence/v2/jobs/events
MP_MESSAGING_OUTGOING_KOGITO_JOB_SERVICE_JOB_REQUEST_EVENTS_URL

mp_messaging_outgoing_kogito_processinstances_events_url = http://sonataflow-platform-data-index-service.usecase3-persistence/processes
MP_MESSAGING_OUTGOING_KOGITO_PROCESSINSTANCES_EVENTS_URL

mp_messaging_outgoing_kogito_processdefinitions_events_url = http://sonataflow-platform-data-index-service.usecase3-persistence/definitions
MP_MESSAGING_OUTGOING_KOGITO_PROCESSDEFINITIONS_EVENTS_URL

org_kie_kogito_addons_knative_eventing_health_enabled = false
ORG_KIE_KOGITO_ADDONS_KNATIVE_EVENTING_HEALTH_ENABLED



	"application-prod.properties": "kogito.data-index.health-enabled = true
		kogito.data-index.url = http://sonataflow-platform-data-index-service.usecase3-persistence
		kogito.events.processdefinitions.enabled = true
		kogito.events.processdefinitions.errors.propagate = true
		kogito.events.processinstances.enabled = true
		kogito.events.usertasks.enabled = false
		kogito.jobs-service.health-enabled = true
		kogito.jobs-service.url = http://sonataflow-platform-jobs-service.usecase3-persistence
		kogito.persistence.proto.marshaller = false
		kogito.persistence.type = jdbc
		mp.messaging.outgoing.kogito-job-service-job-request-events.connector = quarkus-http
		mp.messaging.outgoing.kogito-job-service-job-request-events.url = http://sonataflow-platform-jobs-service.usecase3-persistence/v2/jobs/events
		mp.messaging.outgoing.kogito-processdefinitions-events.url = http://sonataflow-platform-data-index-service.usecase3-persistence/definitions
		mp.messaging.outgoing.kogito-processinstances-events.url = http://sonataflow-platform-data-index-service.usecase3-persistence/processes
		org.kie.kogito.addons.knative.eventing.health-enabled = false
		quarkus.datasource.db-kind = postgresql
		kogito.service.url = http://callbackstatetimeouts.usecase3-persistence
		quarkus.http.port = 8080
		quarkus.http.host = 0.0.0.0
		quarkus.devservices.enabled = false
		quarkus.kogito.devservices.enabled = false


