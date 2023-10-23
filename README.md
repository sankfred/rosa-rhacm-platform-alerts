# Update alertmanager secret

oc -n open-cluster-management-observability create secret generic alertmanager-config --from-file=alertmanager.yaml --dry-run=client -o=yaml |  oc -n open-cluster-management-observability replace secret --filename=-

# Amtool reference
https://cloud.redhat.com/blog/how-to-use-amtool-to-manage-red-hat-advanced-cluster-management-for-kubernetes-alerts

# Query alerts

oc rsh observability-alertmanager-0 amtool alert query --alertmanager.url http://localhost:9093

# Query specific alerts

oc rsh observability-alertmanager-0 amtool alert query alertname="PlatformAlert" --alertmanager.url http://localhost:9093 -o json|jq -r .[]

# Prometheus Query (PromQL) Docs

https://prometheus.io/docs/prometheus/latest/querying/basics/