# Isolated Sage API group

This fork uses `sage.opensearch.org/v1` for its ten custom resource types, including
`OpenSearchCluster` and `OpensearchActionGroup`. The CRDs have distinct names, such
as `opensearchclusters.sage.opensearch.org` and
`opensearchactiongroups.sage.opensearch.org`. The existing
`opensearch.org` and `opensearch.opster.io` CRDs remain owned by the original
Helm release.

This is intended for a **new** operator installation, not an in-place upgrade of
an existing release. Build and publish this fork's operator image first: the
standard `opensearchproject/opensearch-operator` image does not watch the Sage
API group.

```sh
helm install sage ./charts/opensearch-operator \
  --namespace sage --create-namespace \
  --set manager.image.repository=<your-fork-image> \
  --set manager.image.tag=<your-fork-tag> \
  --set legacyAPI.enabled=false
```

The second release installs its own Sage CRDs. Keep the first release and its
CRDs in place. Create new clusters with the Sage group:

```yaml
apiVersion: sage.opensearch.org/v1
kind: OpenSearchCluster
metadata:
  name: sage-cluster
  namespace: sage
spec:
  general:
    serviceName: sage-cluster
    version: "3"
  nodePools:
    - component: masters
      replicas: 3
      diskSize: 5Gi
      roles: [cluster_manager, data]
```

The cluster Helm chart also defaults to the Sage group in this fork. Set
`manager.watchNamespace` to the namespaces the second operator should manage
if you do not want it to watch all namespaces. The API group isolates the
custom resources, while a distinct namespace and resource names keep the
underlying Services, StatefulSets, PVCs, and Secrets separate.

Legacy API support is disabled by default. Do not enable it when the old
`opensearch.opster.io` CRDs belong to another Helm release. This fork does not
automatically move existing `opensearch.org` custom resources into the Sage
group; copying a resource manifest would require a separate plan for its
existing pods, PVCs, owner references, and data.
