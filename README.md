# Openshift RHACM platform alerts from ROSA managed Clusters 

__Problem:__

ROSA customers are prevented from reporting prometheus platform alerts via alertmanager. The ROSA SRE team prevents updates (hive) to the Openshift Monitoring stack.  RHACM's observability operator is also prevented from updating the cluster-monitoring-config in openshift-monitoring namespace.  

__Solution:__

Engineering is actively working on migrating the ROSA managed cluster-monitoring-config configmap to a cluster resource in a future release.
https://issues.redhat.com/browse/OBSDA-529

__Workaround:__ 

On the RHACM cluster open-cluster-management-observability namespace, create a custom thanos platform alert query. Route alerts via a master alertmanager config.

*NOTE:* the example query will only return an alert metric for notification.  Alert details need investigating by the responsible party.

## Update alertmanager secret

oc -n open-cluster-management-observability create secret generic alertmanager-config --from-file=alertmanager.yaml --dry-run=client -o=yaml |  oc -n open-cluster-management-observability replace secret --filename=-

## Amtool reference
https://cloud.redhat.com/blog/how-to-use-amtool-to-manage-red-hat-advanced-cluster-management-for-kubernetes-alerts

## Query alerts

oc rsh observability-alertmanager-0 amtool alert query --alertmanager.url http://localhost:9093

## Query specific alerts

oc rsh observability-alertmanager-0 amtool alert query alertname="PlatformAlert" --alertmanager.url http://localhost:9093 -o json|jq -r .[]

## Prometheus Query (PromQL) Docs

https://prometheus.io/docs/prometheus/latest/querying/basics/