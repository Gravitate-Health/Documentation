# Istio Deployment

Istio can be installed using a standalone installer, Helm charts, and Operators. For this project we will use the installer as it is the recommended path for production enviromnents.

Steps: 

1. Download istio

```bash
curl -L https://istio.io/downloadIstio | sh -

# or for a specific version (this README was written using 1.20.2)
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.2 TARGET_ARCH=x86_64 sh - 

# Add Istio to your path so it will be easier to use.
cd istio-1.20.2/
export PATH=$PWD/bin:$PATH
```

2. Install Istio operator with the [Configuration file](https://github.com/Gravitate-Health/istio/blob/main/kubernetes/base/001_istio-operator.yaml)

```bash
istioctl install --set profile=demo -y -f kubernetes/base/001_istio-operator.yaml
```

3. Prepare the `default` namespace for injection
```bash
kubectl label namespace default istio-injection=enabled
```

4. Set up the gateway. We have two options, depending on wheter we have a DNS name or not. For only IP access use [this file](https://github.com/Gravitate-Health/istio/blob/main/kubernetes/base/002_gh-gateway-ip.yaml). For this example we will use the one [with domain name and certificate](https://github.com/Gravitate-Health/istio/blob/main/kubernetes/base/002_gh-gateway.yaml):

```bash
kubectl apply -f ./kubernetes/base/002_gh-gateway.yaml
```

5. Patch the IstioIngess service to log the original IP of evry request, for better monitoring and debugging:
```bash
kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec":{"externalTrafficPolicy":"Local"}}'
```

Istio should be installed and working as of this point. You can check everything has deployed correctly running the following command:

```bash
kubectl get all -n istio-system
```
```bash
NAME                                       READY   STATUS    RESTARTS   AGE
pod/istio-egressgateway-85649899f8-t9zf7   1/1     Running   0          3m30s
pod/istio-ingressgateway-f56888458-bbhw9   1/1     Running   0          3m30s
pod/istiod-64848b6c78-mbj9c                1/1     Running   0          3m30s

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                                                      AGE
service/istio-egressgateway    ClusterIP      10.233.48.106   <none>           80/TCP,443/TCP                                                               3m30s
service/istio-ingressgateway   LoadBalancer   10.233.33.24    85.215.199.205   15021:31449/TCP,80:30371/TCP,443:31915/TCP,31400:32682/TCP,15443:32093/TCP   3m30s
service/istiod                 ClusterIP      10.233.50.112   <none>           15010/TCP,15012/TCP,443/TCP,15014/TCP                                        3m30s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-egressgateway    1/1     1            1           3m30s
deployment.apps/istio-ingressgateway   1/1     1            1           3m30s
deployment.apps/istiod                 1/1     1            1           3m30s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-egressgateway-85649899f8   1         1         1       3m30s
replicaset.apps/istio-ingressgateway-f56888458   1         1         1       3m30s
replicaset.apps/istiod-64848b6c78                1         1         1       3m30s
```

1. Set up for Let's Encrypt certificate (only if the gateway with domain name was applied, as Let's Encrypt does not issue certificate for IP addresses). First we have to set the ClusterIssuer resource, and then the certificate, so the clusterIssuer starts de challenge for Let's Encrypt.

Install Istio CertManager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```

Set up the [cluster issuer](https://github.com/Gravitate-Health/istio/blob/main/kubernetes/base/003_cluster-issuer.yaml) (currently using http01 challenge)
```bash
kubectl apply -f ./kubernetes/base/003_cluster-issuer.yaml
```

Instanciate the [certificate](https://github.com/Gravitate-Health/istio/blob/main/kubernetes/base/004_letsencrypt-cert.yaml)
```bash
kubectl apply -f ./kubernetes/base/004_letsencrypt-cert.yaml
```
