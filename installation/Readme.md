# Flow

- `oc new-project open-cluster-management`
- `oc create -f operator-group.yaml`
- `oc create -f subscription.yaml`
- `oc get installplan`
- `oc patch installplan install-4k2q8 --type merge --patch '{"spec":{"approved":true}}' `
- check `search-redis-volume.yaml`
- `oc create -f search-redis-volume.yaml`
- check MCH definitation for `nodeSelector` and `Tolerations`
- `oc create -f mch.yaml`

# Customization for the search volume
https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.5/html/web_console/web-console#search-customization
