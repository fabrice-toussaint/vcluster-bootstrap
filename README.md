Check out [vcluster github](https://github.com/loft-sh/vcluster/blob/main/chart/values.yaml) for the helm chart

Prerequisites:
- Kubernetes v1.28.4
- Helm v3.14.2
- vCluster 0.20.0-beta.1

Create vcluster-parent namespace

```kubectl create namespace vcluster-parent```

Start local docker registry & port-forward:

```
helm upgrade --install local-registry local-registry/ --namespace vcluster-parent
kubectl --namespace vcluster-parent port-forward $POD_NAME $PORT:$CONTAINER_PORT
```

Package and push vcluster-blueprint, [extra information](https://www.vcluster.com/docs/v0.19/deploying-vclusters/init-charts)

```
helm package vcluster-blueprint
helm push vcluster-blueprint-{VERSION}.tgz oci://localhost:$PORT

The local registry setup is currently not working so the following needs to be done:
cat {PATH_TO_PACKAGED_CHART} | base64 | pbcopy
then paste this in the vcluster-child.yaml --> bundle: <base64_encoded_tar_bundle>
```

To upgrade and install vcluster-child
```
helm upgrade --install vcluster-child vcluster \
    --values vcluster-child.yaml \
    --repo https://charts.loft.sh \
    --namespace vcluster-parent \
    --version 0.20.0-beta.1 \
    --repository-config=''
```

To delete
```
helm uninstall local-registry --namespace vcluster-parent
helm uninstall vcluster-child --namespace vcluster-parent
```