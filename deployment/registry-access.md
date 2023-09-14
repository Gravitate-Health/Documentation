# Private image registry from k8s cluster
In order to be able to pull images from a private registry, it is needed that a service account has access to that registry. The process is creating a secret with a registry access token, and patching the service account to read that secret:

```bash
# Create image-pull-secret
kubectl create secret docker-registry image-pull-secret --docker-server=gravitate-registry.cr.de-fra.ionos.com --docker-username=gravitatecluster --docker-password="REGISTRY_ACCESS_TOKEN" -n default

# Patch default service account with image-pull-secret
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "image-pull-secret"}]}' -n default
```
