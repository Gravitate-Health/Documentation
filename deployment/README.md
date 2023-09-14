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

Istio depoyment consists in:
- Downloading Istio
- Configuring Istio
- Deploying Istio
- Request certificate to Let's Encrypt

Refer to the [Istio Readme](./istio-deployment.md).

Once all the components have been installed, the certificate is issued, and traffic can ingress and egress from the cluster, you can go on and configure the image registry.

### Configure image registry and secret

An image registry is needed to store al lthe FOSPS images. After a registry is stablished, the cluster needs an access token to read from a private registry.

Refer to the [registry access Readme](./registry-access.md) to configure the cluster to access the private registry.


### Deploy Keycloak

Keycloak is the identity provider and the first service to be deployed.

Refer to the [Keycloak Readme](./registry-access.md) to deploy and configure Keycloak.


### Deploy FOSPS services

Work in progress.
