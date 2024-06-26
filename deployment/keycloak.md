# Keycloak deployment

To deploy Keycloak, first deploy the database, and then the Keycloak server.

NOTE: All the files reference to the [Keycloak repo](https://github.com/Gravitate-Health/keycloak).

## PostgreSQL Helm Deployment

Postgres is installed via [Bitnami's Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql/).

1. Create PVC and apply it:

```bash
kubectl apply -f YAMLs/002_postgres_pvc.yaml
```


2. Create a `values.yaml` file to override default values. The secret values will be creates as a kubernetes secret.

```yaml
# define default database user, name, and password for PostgreSQL deployment
auth:
  enablePostgresUser: true
  postgresPassword: "ADMIN_STRONG_PASSWORD"
  username: "keycloak"
  password: "KEYCLOAK_DATABASE_PASSWORD" #This password will be used by keycloak
  database: "keycloak"

# The postgres helm chart deployment will be using PVC postgres-data-keycloak
primary:
  persistence:
    enabled: true
    existingClaim: "postgres-data-keycloak" # This is the previously created PVC
```

3. Add Helm repo and install the chart:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install keycloak-helm -f YAMLs/values.yaml bitnami/postgresql
```

ANd check everything is installed correctly:

```bash
kubectl get secrets
```
```bash
NAME                                            TYPE                             DATA   AGE
keycloak-helm-postgresql                        Opaque                           2      1m25s
```

```bash
kubectl get pods
```
```bash
NAME                                    READY   STATUS    RESTARTS   AGE
keycloak-helm-postgresql-0              2/2     Running   0          2m
```

```bash
kubectl get services
```
```bash
NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE

keycloak-helm-postgresql                ClusterIP   10.233.1.115    <none>        5432/TCP            2m43s
keycloak-helm-postgresql-hl             ClusterIP   None            <none>        5432/TCP            2m43s
```

## Keycloak k8s deployment

The Kubernetes deployment uses the public Keycloak image hosted at [quay.io](https://quay.io/repository/keycloak/keycloak). And the public Postgresql image hosted at [hub.docker](https://hub.docker.com/_/postgres), which will be used as persistence. The files for the deployment can be found at the [YAMLs](https://github.com/Gravitate-Health/keycloak/tree/main/YAMLs) directory

Both the deployment files for the Postgres DB and the Keycloak contain several environment variables which can be modified. These environmnet variables are the ones we used but the configuration allows for much more. Furthermore, the file [001_keycloak-secrets.yaml](YAMLs/001_keycloak-secrets.yaml) contains the values for the passwords to be used in the deployment files, you must generate your own and convert it into base64 and replace it. For example:

```bash
echo -n Mypassword1234! | base64 -w 0     # Make sure there is no trailing "\n", it will fail
```

And paste the base64 encoded password to the secret yaml file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keycloak-secret
data:
  adminpass: BASE64_ENCODED_PASSWORD=

```

- Keycloak environment variables

| Environment Variable    | description                                                                                                   | default                                      |
|-------------------------|---------------------------------------------------------------------------------------------------------------|----------------------------------------------|
| KEYCLOAK_ADMIN          | Username of the Keycloak admin user                                                                           | admin                                        |
| KECYLOAK_ADMIN_PASSWORD | Password of the Keycloak admin user                                                                           | \<secret>                                    |
| KC_DB_PASSWORD          | Password of the Postgres DB                                                                                   | \<secret>                                    |
| KC_DB_USERNAME          | Username of the database user                                                                                 | keycloak                                     |
| KC_PROXY                | Type of proxy to be used                                                                                      | edge                                         |
| KC_DB_URL               | Full connection URL to the database                                                                           | jdbc:postgresql://keycloak-postgres/keycloak |
| KC_HOSTNAME_STRICT      | Allow other hostnames to be used, required "false" for use with a reverse proxy without further configuration | false                                        |
| KC_HTTP_ENABLED         | If HTTP can be used                                                                                           | true                                         |
| KC_DB_SCHEMA            | Schema of the database                                                                                        | public                                       |
| KC_LOG_LEVEL            | Log level                                                                                                     | debug                                        |


Once the database is ready the Keycloak can be deployed:

```bash
kubectl apply -f YAMLs/005_keycloak-service.yaml
kubectl apply -f YAMLs/006_keycloak-deployment.yaml
```

You can check if the deployment is ready by running:

```bash
kubectl get pod | grep "keycloak"
```
```bash
NAMESPACE            NAME                                         READY   STATUS    RESTARTS        AGE
<namespace>          keycloak-54d87cb874-fgfqt                    1/1     Running   0               12d
```

If the pod is ready you can access the service by other services in the same namespace by using the name of its Kubernetes service and the port (especified in [005_keycloak-service.yaml](https://github.com/Gravitate-Health/keycloak/tree/main/YAMLs)). You can also obtain both by running the following commands:

```bash
kubectl get svc | grep "keycloak"
```
```bash
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
keycloak                   ClusterIP   10.152.183.207   <none>        8080/TCP            48d
```

The type of the service is _ClusterIP_ which means that the service can only be accessed from inside the cluster. Moreover, if the Kubernetes cluster has a DNS manager other services can access services in other namespaces using the following URL: ```http://<service-name>.<namespace>.svc.cluster.local```. To learn more about the types of services and its uses in Kubernetes, here is the [official documentation](https://kubernetes.io/docs/concepts/services-networking/). 

## Apply istio's VirtualService

In order to be accesible from the internet, the VirtualService must be configured for Istio. Apply the [following yaml file](https://github.com/Gravitate-Health/keycloak/blob/main/YAMLs/007_keycloak-vs.yaml) in the [YAMLs folder for keycloak](https://github.com/Gravitate-Health/keycloak/tree/main/YAMLs)

```bash
kubectl apply -f 007_keycloak-vs.yaml
```

## Create service user

Create a user for the keycloak-registration service to be able to manage users:

- Create a keycloak user with the email and password specified in the secrets file of the keycloak repository.
- Edit user's roles going to Role Mappings --> Client Roles = `realm-management` --> Assign the role `manage-users`

## Email settings

If you need Keycloak to send emails (Email verification, password recovery, etc.), you need to configure it.
- 1. The "admin" account of the "master" realm needs to have an email configured.
- 2. Go to the realm settings where FOSPS users are located, let's assume 'GravitatateHealth' realm.
- 3. Go to the 'email' tab.
- 4. Set the email server configuration.
- 5. Use the 'Test connection' button to try the configuration.