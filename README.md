Check out [vcluster github](https://github.com/loft-sh/vcluster/blob/main/chart/values.yaml) for the helm chart

Prerequisites:
- Kubernetes v1.28.4
- Helm v3.14.2
- vCluster 0.20.0-beta.1

Setting up a private local Docker registry:
```
kubectl create namespace local-registry
helm dependency update --namespace local-registry local-registry/
helm upgrade --install local-registry local-registry/ --namespace local-registry
```
When this runs successfully it is necessary to copy the created certificate and make sure that it is trusted by the [filesystem](https://support.securly.com/hc/en-us/articles/206058318-How-to-install-the-Securly-SSL-certificate-on-Mac-OSX).
```
kubectl -n local-registry get secrets/self-signed-certs-secret -o json -o=jsonpath="{.data.tls\.crt}" | base64 -d
```
After this add local-registry.com to etc/hosts and point to 127.0.0.1 by adding the following line:
```
127.0.0.1       local-registry.com
```

This makes it possible to package and push the [vcluster](https://www.vcluster.com/docs/v0.19/deploying-vclusters/init-charts)-blueprint to the local private registry:
```
helm package vcluster-blueprint
helm push vcluster-blueprint-{VERSION}.tgz oci://local-registry.com
```
To upgrade and install vcluster-child
```
kubectl create namespace vcluster-parent
helm upgrade --install vcluster-child vcluster \
    --values vcluster-child.yaml \
    --repo https://charts.loft.sh \
    --namespace vcluster-parent \
    --version 0.20.0-beta.1 \
    --repository-config=''
```
To delete
```
helm uninstall local-registry --namespace local-registry
helm uninstall vcluster-child --namespace vcluster-parent
```