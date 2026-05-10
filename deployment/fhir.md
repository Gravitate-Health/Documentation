# FHIR deployment

This documentation covers the deployment of both FHIR servers used by the Gravitate-Health platform:

- **fhir-epi** — HAPI FHIR R5 server for electronic Product Information (ePI) resources, served at `/epi/api`
- **fhir-ips** — HAPI FHIR R4 server with IPS support for International Patient Summaries, served at `/ips/api`

Both servers are deployed using the Helm charts in the [hapi-fhir-deployments](https://github.com/Gravitate-Health/hapi-fhir-deployments) repository. Each chart bundles PostgreSQL (Bitnami), the HAPI FHIR JPA Server (official upstream chart), an Istio VirtualService, and a post-install Job that patches the probe paths automatically.

> **Note:** The previously separate `fhir-epi` and `fhir-ips` repositories (which built custom Docker images) have been archived. The current approach uses the official `hapiproject/hapi` image configured entirely via environment variables.

## Prerequisites

- Kubernetes ≥ 1.24, Helm ≥ 3.10
- Istio installed with the `gh-gateway` Gateway already deployed (see [istio repository](https://github.com/Gravitate-Health/istio))

## Installation

### 1. Add Helm repositories

```bash
helm repo add hapifhir https://hapifhir.github.io/hapi-fhir-jpaserver-starter/
helm repo add bitnami  https://charts.bitnami.com/bitnami
helm repo update
```

### 2. Clone the deployment repository

```bash
git clone https://github.com/Gravitate-Health/hapi-fhir-deployments.git
cd hapi-fhir-deployments
```

### 3. Download chart dependencies

```bash
helm dependency update charts/fhir-epi
helm dependency update charts/fhir-ips
```

### 4. Deploy — ePI FHIR server

```bash
# Create DB credentials secret
kubectl create secret generic postgresql-fhir-epi \
  --from-literal=postgres-password=<admin-password> \
  --from-literal=password=<app-password>

# Install the chart
helm install fhir-epi charts/fhir-epi \
  --set postgresql.auth.existingSecret=postgresql-fhir-epi \
  --set hapi.externalDatabase.existingSecret=postgresql-fhir-epi \
  --set hapi.externalDatabase.existingSecretKey=password
```

### 5. Deploy — IPS FHIR server

```bash
# Create DB credentials secret
kubectl create secret generic postgresql-fhir-ips \
  --from-literal=postgres-password=<admin-password> \
  --from-literal=password=<app-password>

# Install the chart
helm install fhir-ips charts/fhir-ips \
  --set postgresql.auth.existingSecret=postgresql-fhir-ips \
  --set hapi.externalDatabase.existingSecret=postgresql-fhir-ips \
  --set hapi.externalDatabase.existingSecretKey=password
```

### Upgrade

```bash
helm upgrade fhir-epi charts/fhir-epi [--set ...]
helm upgrade fhir-ips charts/fhir-ips [--set ...]
```

The probe-patch Job runs automatically on every install and upgrade — no manual `kubectl patch` is needed.

## Verifying the deployment

```bash
kubectl get all | grep -E "fhir-(epi|ips)|postgresql-fhir"
```

Expected output:
```
pod/fhir-server-epi-<hash>        2/2     Running   0   ...
pod/fhir-epi-postgresql-0         2/2     Running   0   ...
pod/fhir-server-ips-<hash>        2/2     Running   0   ...
pod/fhir-ips-postgresql-0         2/2     Running   0   ...

service/fhir-server-epi           ClusterIP   ...   8080/TCP,8081/TCP
service/fhir-epi-postgresql       ClusterIP   ...   5432/TCP
service/fhir-server-ips           ClusterIP   ...   8080/TCP,8081/TCP
service/fhir-ips-postgresql       ClusterIP   ...   5432/TCP
```

## Endpoints

| Server | Public URL (via Istio) | Internal service |
|--------|----------------------|-----------------|
| ePI FHIR API | `https://<dns>/epi/api/fhir/` | `fhir-server-epi:8080` |
| IPS FHIR API | `https://<dns>/ips/api/fhir/` | `fhir-server-ips:8080` |
| IPS Summary op | `https://<dns>/ips/api/fhir/Patient/<id>/$summary` | — |

## Key differences between ePI and IPS servers

| Property | fhir-epi | fhir-ips |
|----------|----------|----------|
| FHIR version | R5 | R4 |
| Context path | `/epi/api` | `/ips/api` |
| IPS operation | No | Yes |
| Implementation Guide | Gravitate-Health GH 0.1.0 | HL7 IPS 2.0.0-ballot |
| Default image tag | `hapiproject/hapi:v7.6.0` | `hapiproject/hapi:v7.4.0` |

## Full configuration reference

See the chart-specific READMEs in the deployment repository:

- [charts/fhir-epi/README.md](https://github.com/Gravitate-Health/hapi-fhir-deployments/blob/main/charts/fhir-epi/README.md)
- [charts/fhir-ips/README.md](https://github.com/Gravitate-Health/hapi-fhir-deployments/blob/main/charts/fhir-ips/README.md)

