# Gravitate-Health FOSPS Deployment Documentation

This README will guide the user through the process of deploying FOSPS, with all tghe requirements and needed steps.Through the course of the project, there will these options to deploy FOSPS:
- Manually to a existing k8s cluster.
- Helm chart (Coming Soon)

## Native kubernetes deployment

The requirements for FOSPS are:
- Existing k8s cluster (developed in native kubernetes but should work in any copmlinat distribution)
- Minimum Hardware resources (acarross the cluster): 16 GB RAM, 4 vCPUs.

The deployment of FOSPS is done in different steps:
- Deploy Istio.
- Configure Registry and image-pull-secret
- Deploy Keycloak
- Deploy FOSPS services

### Deploy Istio
---

Istio depoyment consists in:
- Downloading Istio
- Configuring Istio
- Deploying Istio
- Request certificate to Let's Encrypt

Refer to the [Istio Readme](./istio-deployment.md).

Once all the components have been installed, the certificate is issued, and traffic can ingress and egress from the cluster, you can go on and configure the image registry.

### Configure image registry and secret
---

An image registry is needed to store al lthe FOSPS images. After a registry is stablished, the cluster needs an access token to read from a private registry.

Refer to the [registry access Readme](./registry-access.md) to configure the cluster to access the private registry.


### Deploy Keycloak
---

Keycloak is the identity provider and the first service to be deployed.

Refer to the [Keycloak Readme](./registry-access.md) to deploy and configure Keycloak.

### Deploy FHIR server(s) (Helm chart)
---

Currently there are two separate instances for FHIR: ePI and IPS server:

Refer to the [FHIR Readme](./fhir.md) to deploy and configure FHIR servers.


### Deploy FOSPS services
---

Work in progress.

### Deploy APIs & other FOSPS services
---

With Istio, keycloak and the registry, we have all set up to deploy the FOSPS services:

List of services to deploy:
- Focusing:
  - [Focusing Manager](https://github.com/Gravitate-Health/focusing-manager)
  - [Lens Selector](https://github.com/Gravitate-Health/lens-selector-mvp2)
  - [Manual Preprocessor](https://github.com/Gravitate-Health/preprocessing-service-manual)
  - [Automatic Preprocessor](https://github.com/Gravitate-Health/preprocessing-service-mvp2)
- FHIR:
  - [FHIR ePI](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-epi)
  - [FHIR IPS](https://github.com/Gravitate-Health/hapi-fhir-jpaserver-starter-ips)
  - [Fhir Connector](https://github.com/Gravitate-Health/fhir-connector)
- Other:
  - [Swagger UI](https://github.com/Gravitate-Health/swagger-deployment)
  - [Keycloak Registration](https://github.com/Gravitate-Health/keycloak-registration)
  - [Therminology shortlist](https://github.com/Gravitate-Health/terminology-service)

For each service, you will find a folder in each repository called `kubernetes`