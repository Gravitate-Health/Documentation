# Istio Deployment

Istio can be installed using a standalone installer, Helm charts, and Operators. For this project we will use the installer as it is the recommended path for production enviromnents.

Steps: 

1. Download istio

```bash
curl -L https://istio.io/downloadIstio | sh -

# or for a specific version (this README was written using 1.16.1)
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.16.1 TARGET_ARCH=x86_64 sh - 

# Add Istio to your path so it will be easier to use.
cd istio-1.16.1/
export PATH=$PWD/bin:$PATH
```

2. Install Istio operator. Keep in mind that depending on your platform you might need to prepare it before the Istio installation (https://istio.io/latest/docs/setup/platform-setup/). It is installed with [istio-operator.yml](./fosps-enviroment/001_istio-operator.yaml)

```bash
istioctl apply -f ./fosps-enviroment/001_istio-operator.yaml

# Alternatively it can be written directly in the comman
istioctl install --set profile=default -y --set components.egressGateways[0].name=istio-egressgateway --set components.egressGateways[0].enabled=true --set meshConfig.accessLogFile=/dev/stdout    # Use 'default' profile for production environments 
```

2. Prepare the `default` namespace for injection
```bash
kubectl label namespace default istio-injection=enabled
```

3. Set up the gateway. We have two options, depending on wheter we have a DNS name or not. For only IP access use [this file](./fosps-enviroment/002_gh-gateway-ip.yaml). For this example we will use the one [with domain name and certificate](./fosps-enviroment/002_gh-gateway.yaml):

```bash
kubectl apply -f ./fosps-enviroment/002_gh-gateway.yaml
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

4. Set up for Let's Encrypt certificate (only if the gateway with domain name was applied, as Let's Encrypt does not issue certificate for IP addresses). First we have to set the ClusterIssuer resource, and then the certificate, so the clusterIssuer starts de challenge for Let's Encrypt.

Install Istio CertManager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```

Set up the [cluster issuer](./fosps-enviroment/003_cluster-issuer.yaml) (currently using http01 challenge)
```bash
istioctl apply -f ./fosps-enviroment/003_cluster-issuer.yaml
```

Instanciate the [certificate](./fosps-enviroment/004_letsencrypt-cert.yaml)
```bash
istioctl apply -f ./fosps-enviroment/004_letsencrypt-cert.yaml
```