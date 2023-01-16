# Flow

- Check labels of the node and existance of the MCPs
- Check defaultNodeSelector `oc get scheduler/cluster -o yaml`
- Move components to "Infra" nodes:
  - Router: 
    - Check with `oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.nodePlacement}{"\n"}'` 
    - Patch with `oc patch ingresscontroller/default -n  openshift-ingress-operator  --type=merge -p '{"spec":{"nodePlacement": {"nodeSelector": {"matchLabels": {"node-role.kubernetes.io/infra": ""}}' `
  - Registry:
    - Check with `oc get configs.imageregistry.operator.openshift.io/cluster -o jsonpath='{.spec.nodeSelector}{"\n"}'`
    - Patch with `oc patch configs.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"nodeSelector": {"node-role.kubernetes.io/infra": ""}' `
  - Monitoring:
    - Check with `oc get cm/cluster-monitoring-config -n openshift-monitoring -o yaml`
    - Edit ConfigMap and add `nodeSelector` for all relevant components, refer to the [documentation](https://docs.openshift.com/container-platform/4.11/monitoring/configuring-the-monitoring-stack.html#moving-monitoring-components-to-different-nodes_configuring-the-monitoring-stack)
  - Logging:
    - Refer to the [documentation](https://docs.openshift.com/container-platform/4.11/machine_management/creating-infrastructure-machinesets.html?extIdCarryOver=true&sc_cid=701f2000001OH6fAAG#infrastructure-moving-logging_creating-infrastructure-machinesets)
- Check what still remains on the "Worker" nodes
- Remove nodes from Cluster, following the [documentation](https://docs.openshift.com/container-platform/4.11/nodes/nodes/nodes-nodes-working.html#nodes-nodes-working-deleting-bare-metal_nodes-nodes-working)

For reference, this [Red Hat Solution](https://access.redhat.com/solutions/5034771) has more details.

