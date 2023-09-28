# FHIR deployment

This documentation refers to FHIR IPS server. The same procedure must be done to deploy FHIR ePI server, but changing the values.

## PostgreSQL Helm Deployment

First of all we will need to deply the database that our FHIR IPS Server will use. We will use [Bitnami PostgreSQL](https://github.com/bitnami/charts/tree/main/bitnami/postgresql/) Helm chart to deploy the database. This will be done by creating a PVC and then deploying the Helm chart. We have an example of PVC in [kubernetes/002_postgres-pvc.yaml](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-ips/blob/master/kubernetes/002_postgres-pvc.yaml). Without this, the database won't be persistent and will be deleted when the pod is deleted.

```bash
kubectl apply -f kubernetes/002_postgres-pvc.yml
```

After this, we need to create a secret with the password for the database. We have an example in [kubernetes/003_postgres-secret.yaml](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-ips/blob/master/kubernetes/003_postgres-secret.yaml). You should change the password in this file, and encode the password in Base64. This password will be used in the Helm chart. Generate the password with the following command:

```bash
echo "YOUR_PASSWORD" | base64 -w 0
```

Then, apply the secret with the following command:

```bash
kubectl apply -f kubernetes/003_postgres-secret.yml
```

Finally, we can deploy the Helm chart. We will use the Bitnami's Chart and a values file with the configuration. We have an example in [kubernetes/values.yaml](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-ips/blob/master/kubernetes/values.yaml). Edit the values file to change the password and the PVC name. Then, deploy the Helm chart with the following command:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --render-subchart-notes hapi-fhir-jpaserver-ips bitnami/postgresql --values=kubernetes/values.yaml
```

## HAPI FHIR Helm Deployment

Following the instructions in the [Helm Chart](https://github.com/hapifhir/hapi-fhir-jpaserver-starter/tree/master/charts/hapi-fhir-jpaserver) documentation porceed with the installation. The [values.yaml](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-ips/blob/master/charts/hapi-fhir-jpaserver/values.yaml) file has been modified for the Gravitate Health platform, you can find the full list of variables in the original repository, below is a list of the changes made:

| Variable                       | value       |
|--------------------------------|-------------|
|path| /ips/api|
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
LAST DEPLOYED: Thu Sep 28 12:13:59 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=fhir-server-ips,app.kubernetes.io/instance=hapi-fhir-jpaserver-ips" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
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
kubectl apply -f kubernetes/001_fhir-ips-vs.yml
```

If the pods are ready you can access the service by other services in the same namespace by using the name of its Kubernetes service and the port. Moreover, if the Kubernetes cluster has a DNS manager other services can access services in other namespaces using the following URL: ```http://<service-name>.<namespace>.svc.cluster.local```. To learn more about the types of services and its uses in Kubernetes, here is the [official documentation](https://kubernetes.io/docs/concepts/services-networking/). Alternatively if the [Istio service mesh](https://github.com/Gravitate-Health/istio) has been deployed, the service will be proxied to the outside of the cluster at `https://<DNS>/ips/api/fhir/`.
