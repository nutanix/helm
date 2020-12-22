# Karbon metric server Helm chart

This is the helm chart tailored for Nutanix Karbon

# Kubernetes Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes
built-in autoscaling pipelines.

Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through [Metrics API] 
for use by [Horizontal Pod Autoscaler] and [Vertical Pod Autoscaler]. Metrics API can also be accessed by `kubectl top`,
making it easier to debug autoscaling pipelines.

Metrics Server is not meant for non-autoscaling purposes. For example, don't use it to forward metrics to monitoring solutions, or as a source of monitoring solution metrics.

Metrics Server offers:
- A single deployment that works on most clusters
- Scalable support up to 5,000 node clusters
- Resource efficiency: Metrics Server uses 1m core of CPU and 3 MB of memory per node

For the detail info, please visit below official git repo
https://github.com/kubernetes-sigs/metrics-server

## Prerequisites

- Kubernetes 1.13 or later
- Created base on metrics server 0.3.7 version
- Metrics API group/version(metrics.k8s.io/v1beta1)
- Karbon 2.1.x or later

## Installing the Chart

To install the chart with the name `metric-server`:

```console
helm repo add nutanix https://nutanix.github.io/helm/

helm install --name metric-server nutanix/metric-server
```

## Uninstalling the Chart

To uninstall/delete the `metric-server` deployment:

```console
$ helm delete metric-server
```

## Modified Configuration for Karbon deployment

```console
    deployment.yaml
        ...
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP
        ...
``