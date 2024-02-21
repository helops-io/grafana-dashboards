# grafana-dashboards <!-- omit in toc -->

## Table of contents <!-- omit in toc -->

- [Description](#description)
- [Features](#features)
- [Dashboards](#dashboards)
- [Installation](#installation)
  - [Install manually](#install-manually)
  - [Install via grafana.com](#install-via-grafanacom)
  - [Install with ArgoCD](#install-with-argocd)
  - [Install with Helm values](#install-with-helm-values)
  - [Install as ConfigMaps](#install-as-configmaps)
  - [Install as ConfigMaps with Terraform](#install-as-configmaps-with-terraform)
- [Known issue(s)](#known-issues)
  - [Broken panels due to a too-high resolution](#broken-panels-due-to-a-too-high-resolution)
  - [Broken panels on k8s-views-nodes due to the nodename label](#broken-panels-on-k8s-views-nodes-due-to-the-nodename-label)
- [Contributing](#contributing)

## Description

This repository contains a modern set of [Grafana](https://github.com/grafana/grafana) dashboards for [Kubernetes](https://github.com/kubernetes/kubernetes).\
They are inspired by many other dashboards from `kubernetes-mixin` and `grafana.com`.

More information about them in my article: [A set of modern Grafana dashboards for Kubernetes](https://0xdc.me/blog/a-set-of-modern-grafana-dashboards-for-kubernetes/)

You can also download them on [Grafana.com](https://grafana.com/grafana/dashboards/?plcmt=top-nav&cta=downloads&search=helops-io).

## Features

These dashboards are made and tested for the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) chart, but they should work well with others as soon as you have [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) and [prometheus-node-exporter](https://github.com/prometheus/node_exporter) installed on your Kubernetes cluster.

They are not backward compatible with older Grafana versions because they try to take advantage of Grafana's newest features like:

- `gradient mode` introduced in Grafana 8.1 ([Grafana Blog post](https://grafana.com/blog/2021/09/10/new-in-grafana-8.1-gradient-mode-for-time-series-visualizations-and-dynamic-panel-configuration/))
- `time series` visualization panel introduced in Grafana 7.4 ([Grafana Blog post](https://grafana.com/blog/2021/02/10/how-the-new-time-series-panel-brings-major-performance-improvements-and-new-visualization-features-to-grafana-7.4/))
- `$__rate_interval` variable introduced in Grafana 7.2 ([Grafana Blog post](https://grafana.com/blog/2020/09/28/new-in-grafana-7.2-__rate_interval-for-prometheus-rate-queries-that-just-work/))

They also have a `Prometheus Datasource` variable so they will work on a federated Grafana instance.

As an example, here's how the `Kubernetes / Views / Overall Summary` dashboard looks like:

![screenshot](https://raw.githubusercontent.com/helops-io/docs-assets/main/grafana-dashboards/k8s-overall-summary.png "Kubernetes Overall Summary View Screenshot")

## Dashboards

| File name                  | Description | Screenshot |
|:---------------------------|:------------|:----------:|
| k8s-addons-prometheus.json | `Global` Overall Summary . | [LINK]() |
| k8s-addons-trivy-operator.json | Dashboard for the Trivy Operator from Aqua Security. | [LINK]() |
| k8s_system_api.json | Dashboard for the API Server Kubernetes component. | [LINK]() |
| k8s-system-coredns.json    | Show information on the CoreDNS Kubernetes component. | [LINK]() |
| k8s-views-global.json      | `Global` level view dashboard for Kubernetes. | [LINK]() |
| k8s-views-namespaces.json  | `Namespaces` level view dashboard for Kubernetes. | [LINK]() |
| k8s-views-nodes.json       | `Nodes` level view dashboard for Kubernetes. | [LINK]() |
| k8s-views-pods.json        | `Pods` level view dashboard for Kubernetes. | [LINK]() |

## Installation

In most installation cases, you will need to clone this repository (or your fork):

```terminal
git clone https://github.com/helops-io/grafana-dashboards.git
cd grafana-dashboards
```

If you plan to deploy these dashboards using [ArgoCD](#install-with-argocd), [ConfigMaps](#install-as-configmaps) or [Terraform](#install-as-configmaps-with-terraform), you will also need to enable and configure the `dashboards sidecar` on the Grafana Helm chart to get the dashboards loaded in your Grafana instance:

```yaml
# kube-prometheus-stack values
grafana:
  sidecar:
    dashboards:
      enabled: true
      defaultFolderName: "General"
      label: grafana_dashboard
      labelValue: "1"
      folderAnnotation: grafana_folder
      searchNamespace: ALL
      provider:
        foldersFromFilesStructure: true
```

### Install manually

On the WebUI of your Grafana instance, put your mouse over the `+` sign on the left menu, then click on `Import`.\
Once you are on the Import page, you can upload the JSON files one by one from your local copy using the `Upload JSON file` button.

### Install via grafana.com

On the WebUI of your Grafana instance, put your mouse over the `+` sign on the left menu, then click on `Import`.\
Once you are on the Import page, you can put the grafana.com dashboard ID (see table below) under `Import via grafana.com` then click on the `Load` button. Repeat for each dashboard.

Grafana.com dashboard id list:

| Dashboard                          | ID    |
|:-----------------------------------|:------|
| k8s-overall-summary.json           |       |
| k8s-addons-trivy-operator.json     |       |
| k8s-system-api-server.json         |       |
| k8s-views-global.json              |       |
| k8s-views-nodes.json               |       |
| k8s-views-pods.json                |       |

### Install with ArgoCD

If you have ArgoCD, this will deploy the dashboards in ArgoCD's default project:

```terminal
kubectl apply -f argocd-app.yml
```

You will also need to enable and configure the Grafana `dashboards sidecar` as described in [Installation](#installation).

### Install with Helm values

If you use the official Grafana helm chart or kube-prometheus-stack, you can install the dashboards directly using the `dashboardProviders` & `dashboards` helm chart values.

Depending on your setup, add or merge the following block example to your helm chart values.\
The example is for [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack), for the official [Grafana helm chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana), remove the first line (`grafana:`), and reduce the indentation level of the entire block.

```yaml
grafana:
  # Provision grafana-dashboards
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
      - name: 'grafana-dashboards'
        orgId: 1
        folder: 'Public'
        type: file
        disableDeletion: true
        editable: true
        options:
          path: /var/lib/grafana/dashboards/grafana-dashboards
  dashboards:
    grafana-dashboards:
      k8s-overall-summary:
        url: https://raw.githubusercontent.com/helops-io/grafana-dashboards/master/dashboards/k8s-overall-summary.json
      k8s-system-api:
        url: https://raw.githubusercontent.com/helops-io/grafana-dashboards/master/dashboards/k8s-system-api.json
      k8s-views-global:
        url: https://raw.githubusercontent.com/helops-io/grafana-dashboards/master/dashboards/k8s-views-global.json
      k8s-views-nodes:
        url: https://raw.githubusercontent.com/helops-io/grafana-dashboards/master/dashboards/k8s-views-nodes.json
      k8s-views-pods:
        url: https://raw.githubusercontent.com/helops-io/grafana-dashboards/master/dashboards/k8s-views-pods.json
```

### Install as ConfigMaps

Grafana dashboards can be provisioned as Kubernetes ConfigMaps if you configure the [dashboard sidecar](https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml#L667) available on the official [Grafana Helm Chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana).

To build the ConfigMaps and output them on STDOUT :

```terminal
kubectl kustomize .
```

*Note: no namespace is set by default, you can change that in the `kustomization.yaml` file.*

To build and deploy them directly on your Kubernetes cluster :

```terminal
kubectl apply -k . -n monitoring
```

You will also need to enable and configure the Grafana `dashboards sidecar` as described in [Installation](#installation).

*Note: you can change the namespace if needed.*

### Install as ConfigMaps with Terraform

If you use Terraform to provision your Kubernetes resources, you can convert the generated ConfigMaps to Terraform code using [tfk8s](https://github.com/jrhouston/tfk8s).

To build and convert ConfigMaps to Terraform code :

```terminal
kubectl kustomize . | tfk8s
```

You will also need to enable and configure the Grafana `dashboards sidecar` as described in [Installation](#installation).

*Note: no namespace is set by default, you can change that in the `kustomization.yaml` file.*
