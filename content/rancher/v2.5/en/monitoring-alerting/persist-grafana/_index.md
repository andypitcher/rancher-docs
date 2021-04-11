---
title: Persistent Grafana Dashboards
weight: 4
aliases:
  - /rancher/v2.5/en/monitoring-alerting/v2.5/persist-grafana
---

To allow the Grafana dashboard to persist after the Grafana instance restarts, add the dashboard configuration JSON into a ConfigMap. ConfigMaps also allow the dashboards to be deployed with a GitOps or CD based approach. This allows the dashboard to be put under version control.

- [Creating a Persistent Grafana Dashboard](#creating-a-persistent-grafana-dashboard)
- [Known Issues](#known-issues)

# Creating a Persistent Grafana Dashboard

{{% tabs %}}
{{% tab "Rancher v2.5.8+" %}}
> **Prerequisites:**
> 
> - The monitoring application needs to be installed.
> - You must have at least the **Manage Config Maps** Rancher RBAC permissions assigned to you in the project or namespace that contains the Grafana Dashboards. This correlates to the `monitoring-dashboard-edit` or `monitoring-dashboard-admin` Kubernetes native RBAC Roles exposed by the Monitoring chart.

1. Open the Grafana dashboard. From the **Cluster Explorer,** click **Cluster Explorer > Monitoring.**
1. Log in to Grafana. Note: The default Admin username and password for the Grafana instance is `admin/prom-operator`. (Regardless of who has the password, the **Manage Config Maps** permission in Rancher is still required to access the Grafana instance.) Alternative credentials can also be supplied on deploying or upgrading the chart.
1. Go to the dashboard that you want to persist. In the top navigation menu, go to the dashboard settings by clicking the gear icon.
1. In the left navigation menu, click **JSON Model.**
1. Copy the JSON data structure that appears.
1. Create a ConfigMap in the `cattle-dashboards` namespace. 

    Paste the JSON into the ConfigMap in the format shown in the example below:
      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        labels:
          grafana_dashboard: "1"
        name: <dashboard-name>
        namespace: cattle-dashboards
      data:
        <dashboard-name>.json: |-
          <copied-json>
      ```

      > By default, Grafana is configured to watch all ConfigMaps with the `grafana_dashboard` label within the `cattle-dashboards` namespace.
      >
      > To specify that you would like Grafana to watch for ConfigMaps across all namespaces, refer to [this section.](#configuring-namespaces-for-the-grafana-dashboard-configmap)

**Result:** After the ConfigMap is created, it should show up on the Grafana UI and be persisted even if the Grafana pod is restarted.

Dashboards that are persisted using ConfigMaps cannot be deleted from the Grafana UI. If you attempt to delete the dashboard in the Grafana UI, you will see the error message "Dashboard cannot be deleted because it was provisioned." To delete the dashboard, you will need to delete the ConfigMap.

### Configuring Namespaces for the Grafana Dashboard ConfigMap

To specify that you would like Grafana to watch for ConfigMaps across all namespaces, set:

```
grafana.sidecar.dashboards.searchNamespace=ALL
```

Note that the RBAC roles exposed by the Monitoring chart to add Grafana Dashboards are still restricted to giving permissions for users to add dashboards in the namespace defined in `grafana.dashboards.namespace`, which defaults to `cattle-dashboards`.

{{% /tab %}}
{{% tab "Rancher v2.5.0-v2.5.8" %}}
> **Prerequisites:**
> 
> - The monitoring application needs to be installed.
> - You must have the cluster-admin ClusterRole permission.

1. Open the Grafana dashboard. From the **Cluster Explorer,** click **Cluster Explorer > Monitoring.**
1. Log in to Grafana. Note: The default Admin username and password for the Grafana instance is `admin/prom-operator`. (Regardless of who has the password, cluster administrator permission in Rancher is still required to access the Grafana instance.) Alternative credentials can also be supplied on deploying or upgrading the chart.
1. Go to the dashboard that you want to persist. In the top navigation menu, go to the dashboard settings by clicking the gear icon.
1. In the left navigation menu, click **JSON Model.**
1. Copy the JSON data structure that appears.
1. Create a ConfigMap in the `cattle-dashboards` namespace. The ConfigMap needs to have the label `grafana_dashboard: "1"`. Paste the JSON into the ConfigMap in the format shown in the example below:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      labels:
        grafana_dashboard: "1"
      name: <dashboard-name>
      namespace: cattle-dashboards
    data:
      <dashboard-name>.json: |-
        <copied-json>
	```

**Result:** After the ConfigMap is created, it should show up on the Grafana UI and be persisted even if the Grafana pod is restarted.

Dashboards that are persisted using ConfigMaps cannot be deleted from the Grafana UI. If you attempt to delete the dashboard in the Grafana UI, you will see the error message "Dashboard cannot be deleted because it was provisioned." To delete the dashboard, you will need to delete the ConfigMap.

To prevent the persistent dashboard from being deleted when Monitoring v2 is uninstalled, add the following annotation to the `cattle-dashboards` namespace:

```
helm.sh/resource-policy: "keep"
```

{{% /tab %}}
{{% /tabs %}}

# Known Issues

For users who are using Monitoring V2 v9.4.203 or below, uninstalling the Monitoring chart will delete the `cattle-dashboards` namespace, which will delete all persisted dashboards, unless the namespace is marked with the annotation `helm.sh/resource-policy: "keep"`.

This annotation will be added by default in the new monitoring chart released by Rancher v2.5.8, but it still needs to be manually applied for users of earlier Rancher versions.