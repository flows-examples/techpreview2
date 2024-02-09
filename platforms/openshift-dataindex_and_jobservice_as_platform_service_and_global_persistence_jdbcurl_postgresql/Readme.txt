1) create the namespace

    kubectl create namespace usecase3-platform-persistence-jdbcurl-oc

2) create the SonataFlowPlatform

    kubectl kustomize platforms/openshift-dataindex_and_jobservice_as_platform_service_and_global_persistence_jdbcurl_postgresql | kubectl apply -f - -n usecase3-platform-persistence-jdbcurl-oc


3) wait for DI and JS to start

DataIndex:
    QUARKUS_DATASOURCE_JDBC_URL = jdbc:postgresql://postgres.usecase3-platform-persistence-jdbcurl-oc:5432/sonataflow2

JobsService:

    QUARKUS_DATASOURCE_JDBC_URL = jdbc:postgresql://postgres.usecase3-platform-persistence-jdbcurl-oc:5432/sonataflow2
