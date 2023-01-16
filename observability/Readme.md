# Flow

- `oc create namespace open-cluster-management-observability`
- `DOCKER_CONFIG_JSON=$(oc extract secret/pull-secret -n openshift-config --to=-)`
- 

# Documentation generic
Install the RHACM Observability Stack

Create the namespace:
`oc create namespace open-cluster-management-observability`

Extract the `pullSecret` :
`DOCKER_CONFIG_JSON=$(oc extract secret/pull-secret -n openshift-config --to=-)`

Create the pull secret in the `open-cluster-management-observability` namespace:
`oc create secret generic  multiclusterhub-operator-pull-secret  -n open-cluster-management-observability  --from-literal=.dockerconfigjson="$DOCKER_CONFIG_JSON" --type=kubernetes.io/dockerconfigjson`

Create the OBC:
``` yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: thanos-bc
  namespace: open-cluster-management-observability
spec:
  storageClassName: ocs-external-storagecluster-ceph-rgw
  generateBucketName: observability-bucket
```

Extract the storage configuration details:
```
oc extract configmap/thanos-bc -n open-cluster-management-observability --to=-
# BUCKET_NAME
observability-bucket-eb311c9f-b86e-47ca-8e53-d0ac44e73962
# BUCKET_PORT
8080
# BUCKET_REGION
us-east-1
# BUCKET_SUBREGION
# BUCKET_HOST
rook-ceph-rgw-ocs-external-storagecluster-cephobjectstore.openshift-storage.svc
```

Extract the secret:
```
oc extract secret/thanos-bc -n open-cluster-management-observability --to=-
# AWS_ACCESS_KEY_ID
ODDUF4UU13NP01PKK05Y
# AWS_SECRET_ACCESS_KEY
NedLemnTodoBjwy1T0Fq70YbT85snqDezlU440IF
```

Create the `Secret` for `thanos-object-storage` :

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: thanos-object-storage
  namespace: open-cluster-management-observability
type: Opaque
stringData:
  thanos.yaml: |
    type: s3
    config:
      bucket: BUCKET_NAME
      endpoint: BUCKET_HOST:BUCKET_PORT
      insecure: true
      access_key: AWS_ACCESS_KEY_ID
      secret_key: AWS_SECRET_ACCESS_KEY
```

Create the `MultiClusterObservability` CRD:

``` yaml
kind: MultiClusterObservability
metadata:
  name: observability
spec:
  enableDownsampling: true
  observabilityAddonSpec:
    enableMetrics: true
    interval: 30
  storageConfig:
    alertmanagerStorageSize: 1Gi
    compactStorageSize: 20Gi
    metricObjectStorage:
      key: thanos.yaml
      name: thanos-object-storage
    receiveStorageSize: 20Gi
    ruleStorageSize: 1Gi
    storageClass: ocs-external-storagecluster-ceph-rbd
    storeStorageSize: 10Gi
```

Disable a specific Cluster to not send Metrics anymore:

`oc label managedcluster managed-cluster observability=disabled -n open-cluster-management`

## Customizing the RHACM Observability Stack
Edit the `MultiClusterObservability` CRD to change settings:

`oc edit multiclusterobservability  -n openshift-multicluster-observability`

Example:
``` yaml
# add the following
spec:
  advanced:
    receive:
      replicas: 6
```

Add a custom alerting rule by creating a `ConfigMap` called "thanos-ruler-custom-rules":

``` yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: thanos-ruler-custom-rules
data:
  custom_rules.yaml: |
    groups:
      - name: cluster-health
        rules:
        - alert: ClusterCPUReq-60
          annotations:
            summary: Notify when CPU requests on a cluster are greater than the defined utilization limit
            description: "The cluster {{ $labels.cluster }} has an elevated percentage of CPU requests: {{ $labels.clusterID }}."
          expr: |
            sum(namespace_cpu:kube_pod_container_resource_requests:sum) by (clusterID, cluster) / sum(kube_node_status_allocatable{resource="cpu"}) by (clusterID, cluster) > 0.6
          for: 5s
          labels:
            cluster: "{{ $labels.cluster }}"
            severity: critical
```

Disable the RHACM Observability Stack:

`oc delete multiclusterobservability/observability`
`oc delete namespace open-cluster-management-observability`
