# perses-testing

## Purpose

The purpose of this repo is to test Perses RBAC capabilities. The goal is to give a user access to Perses dashboards within a single project namespace with the minimal permissions necessary. 

## Environment 

OpenShift Cluster 4.18 

OpenShift Cluster Observability Operator 1.2.2


## Enable Perses through Cluster Observability Operator (COO)

Create the following UI Plugin:

```yaml
apiVersion: observability.openshift.io/v1alpha1
kind: UIPlugin
metadata:
  name: monitoring
spec:
  type: Monitoring
  monitoring:
    perses:
      enabled: true
```

## Grant Access to Perses Dashboards to a Single User in a given namespace

The resources in grant-dash-access contain all the resources necessary for giving a user access to dashboards in a given namespace. You will need to change the name of the user it is applied to in the following files:
- clusterrolebinding-user1-reader.yaml
- rolebinding-user1-viewer-dashboard.yaml
- rolebinding-user1-viewer-datasource.yaml
- clusterrolebinding-user1-perses-prometheus-api-editor.yaml

After that you can run the following command

```
oc apply -k grant-dash-access/
```

This grants the user the following permissions:
* ClusterRolebinding: user-reader
	* Gives access to get list watch most OpenShift resources in the cluster
	* **Not REQUIRED for dashboard access. This just lets the user get access to the observe tab in the openshift console. If you don't give the user this rolebinding, you can give them a link to the dashboard and they will still be able to view it**
	* This clusterole created manually, doesn't come with COO Perses and not mentioned as a requirement in the docs
* ClusterRolebinding: perses-prometheus-api-editor
	* Allows user to query prometheus metrics
	* ClusterRole created manually, doesn't come with COO Perses and not mentioned as requirement in the docs
* RoleBinding (namespace scoped): user1-viewer-datasource
	* Allows users to read persesdatasource in a given namespace
	* assigns the ClusterRole persesdatasource-viewer-role which comes installed with COO Perses
* RoleBinding (namespace scoped): user1-viewer-dashboard
	* Allows users to read persesdashboards
	* assigns the ClusterRole persesdashboard-viewer-role which comes installed with COO Perses

## Current Problem/Limitation

Since we want to run this in our production OpenShift environment, we can not just give the user access to query ALL prometheus metrics. This would allow them to see infrastructure metrics and metrics related to other users workloads. We need to figure out a way to make it so that the user only is able to query the metrics that pertain to the dashboard they have access to view, and not all other metrics.