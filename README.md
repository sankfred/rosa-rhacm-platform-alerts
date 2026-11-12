> Original Gist
>
> **[RHACM Alerts on ROSA Gist](https://gist.github.com/05c6b9bbd6352dd213db933fb0de75d2.git)** — Configure Openshift RHACM platform alerts from ROSA managed Clusters.


# Openshift RHACM platform alerts from ROSA managed Clusters 

__Problem:__

ROSA customers are prevented from reporting prometheus platform alerts via alertmanager. The ROSA SRE team prevents updates (hive) to the Openshift Monitoring stack.  RHACM's observability operator is also prevented from updating the cluster-monitoring-config in openshift-monitoring namespace.  

__Solution:__

Engineering is actively working on migrating the ROSA managed cluster-monitoring-config configmap to a custom resource in a future release.
https://issues.redhat.com/browse/OBSDA-529

__Workaround:__ 

On the RHACM cluster open-cluster-management-observability namespace, create a custom thanos rule containing alert query(s). Route alerts via a master alertmanager config.

*NOTE:* the example query will only return an alert metric for notification.  Alert details need investigating by the responsible party.

## Steps 
cluster: RHACM<br>
namespace: open-cluster-management-observability
1. [Write a custom promql alert query. Verify output in grafana.](#step-1-promql-alert-query)
2. [Configure a custom thanos rule containing above query.](#step-2-custom-thanos-rule-configmap)
3. [Configure alertmanager to route query output to 3rd party apps for notification.](#step-3-alertmanager-secret)
4. [Validate routing rules with amtool.](#step-4-validate-routing-rules)

## Step 1: Promql alert query
Write a custom promql alert query. Verify output on the RHACM grafana instance.

Example simple query:
`ALERTS{alertstate="firing",severity!~"info|none"}`

Complex query to include RHACM clusterSet[prod|dev|si] for further classification and routing:<br>
`sum without(clusterID) (label_replace(ALERTS{alertstate="firing",severity="critical"}, "managed_cluster_id", "$1", "clusterID", "(.*)")) * on (managed_cluster_id) group_left(clusterSet) sum(label_replace(acm_managed_cluster_labels, "clusterSet", "$1", "cluster_open_cluster_management_io_clusterset", "(.*)")) without(cluster_open_cluster_management_io_clusterset)`

## Step 2: Custom Thanos Rule ConfigMap
[See the example thanos-ruler-custom-rules.yaml](manifests/thanos-ruler-custom-rules.yaml)

## Step 3: Alertmanager Secret

[See the example alertmanager.yaml](manifests/alertmanager.yaml)

Update secret:<br>
`oc -n open-cluster-management-observability create secret generic alertmanager-config --from-file=alertmanager.yaml=manifests/alertmanager.yaml --dry-run=client -o=yaml |  oc -n open-cluster-management-observability replace secret --filename=-`

## Step 4: Validate routing rules

To confirm the alertmanager config is routing alerts as configured, query the alertmanager pod using 'amtool'

Query all alerts:<br>
`oc rsh observability-alertmanager-0 amtool alert query --alertmanager.url http://localhost:9093`

Query specific alerts:<br>
`oc rsh observability-alertmanager-0 amtool alert query alertname="PlatformAlert" --alertmanager.url http://localhost:9093 -o json|jq -r .[]`

## Reference & Troubleshooting
### Prometheus Query (PromQL) Docs

https://prometheus.io/docs/prometheus/latest/querying/basics/

### Thanos Custom Rules
https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.2/html/observing_environments/observing-environments-intro#customizing-observability

### Alertmanager config

https://prometheus.io/docs/alerting/latest/configuration/

### Amtool reference
https://cloud.redhat.com/blog/how-to-use-amtool-to-manage-red-hat-advanced-cluster-management-for-kubernetes-alerts
