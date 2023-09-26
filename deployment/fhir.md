# FHIR deployment

This documentation refers to FHIR IPS server. The same procedure must be done to deploy FHIR ePI server, but changing the values.

Following the instructions in the [Helm Chart](https://github.com/hapifhir/hapi-fhir-jpaserver-starter/tree/master/charts/hapi-fhir-jpaserver) documentation porceed with the installation. The [values.yaml](gh-values.yaml) file has been modified for the Gravitate Health platform, you can find the full list of variables in the original repository, below is a list of the changes made:

| Variable                       | value       |
|--------------------------------|-------------|
|path| ips/api|
|image.registry| gravitate-registry.cr.de-fra.ionos.com|
|image.repository| hapi-fhir-ips|
|image.tag| "0.0.7"|
|nameOverride| "fhir-server-ips"|
|fullnameOverride| "fhir-server-ips"|
|startupProbe.initialDelaySeconds| 60|
|externalDatabase.host| hapi-fhir-jpaserver-ips-postgresql|
|externalDatabase.user| postgres|

To install run:

```bash
helm repo add hapifhir https://hapifhir.github.io/hapi-fhir-jpaserver-starter/
helm install --render-subchart-notes hapi-fhir-jpaserver-ips hapifhir/hapi-fhir-jpaserver --values=charts/hapi-fhir-jpaserver/values.yaml
```
The response should look like the following:

```
NAME: hapi-fhir-jpaserver-ips
LAST DEPLOYED: Tue Jun 14 10:06:05 2022
NAMESPACE: development
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace development -l "app.kubernetes.io/name=fhir-server,app.kubernetes.io/instance=hapi-fhir-jpaserver" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace development $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace development port-forward $POD_NAME 8080:$CONTAINER_PORT

CHART NAME: postgresql
CHART VERSION: 11.6.2
APP VERSION: 14.3.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    hapi-fhir-jpaserver-postgresql.development.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace development hapi-fhir-jpaserver-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run hapi-fhir-jpaserver-postgresql-client --rm --tty -i --restart='Never' --namespace development --image docker.io/bitnami/postgresql:14.3.0-debian-10-r20 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host hapi-fhir-jpaserver-postgresql -U postgres -d fhir -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace development svc/hapi-fhir-jpaserver-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d fhir -p 5432
```

To check if the resources have been created successfully run:

```bash
kubectl get all | grep "ips"
```
```bash
# Deployments
NAME                                              READY      STATUS    RESTARTS        AGE
pod/fhir-server-ips-75f68b79db-wf89t                2/2     Running           0        43h
pod/fhir-server-ips-7b9cc867c8-p5nnk                2/2     Running           0        42h
pod/hapi-fhir-jpaserver-ips-postgresql-0            2/2     Running           0        42h
# Services
NAME                                                 TYPE        CLUSTER-IP       EXTERNAL-IP                  PORT(S)             AGE
service/fhir-server-ips                         ClusterIP       10.233.2.38            <none>        8080/TCP,9090/TCP             23d
service/hapi-fhir-jpaserver-ips-postgresql      ClusterIP      10.233.38.60            <none>                 5432/TCP             23d
service/hapi-fhir-jpaserver-ips-postgresql-hl   ClusterIP              None            <none>                 5432/TCP             23d
```

NOTE: Sometimes, the deployment builds incorrect liveness, readiness and startup endpoints, so the k8s cluster will always receive 404 from these endpoints, and service will never be ready. You can check this is happening if the pods are periodically restarting, or by running `kubectl describe pod POD_NAME`, wich will report a 404 status on liveness endpoint.

If this is happening, edit manually the deployment `kubectl edit deployment` and delete the livenes, readiness and startup probes. 

### Istio VirtualService deployment

If the [Istio service mesh](https://github.com/Gravitate-Health/istio) has been deployed you can expose the service by running the following command:

```bash
kubectl apply -f kubernetes/fhir_ips_vs.yaml
```

If the pods are ready you can access the service by other services in the same namespace by using the name of its Kubernetes service and the port. Moreover, if the Kubernetes cluster has a DNS manager other services can access services in other namespaces using the following URL: ```http://<service-name>.<namespace>.svc.cluster.local```. To learn more about the types of services and its uses in Kubernetes, here is the [official documentation](https://kubernetes.io/docs/concepts/services-networking/). Alternatively if the [Istio service mesh](https://github.com/Gravitate-Health/istio) has been deployed, the service will be proxied to the outside of the cluster at `https://<DNS>/ips/api/fhir/`.
