# Nutanix CSI plugin Helm chart

## Introduction

The Container Storage Interface (CSI) Volume Driver for Kubernetes leverages Nutanix Volumes and Nutanix Files to provide scalable and persistent storage for stateful applications.

When Files is used for persistent storage, applications on multiple pods can access the same storage, and also have the benefit of multi-pod read and write access.

## Features list

- Nutanix Volumes support
- Nutanix Files support

## Prerequisites

- Kubernetes 1.10 or later
- Kubernetes worker nodes must have the iSCSI package installed (Nutanix Volumes only)

## Installing the Chart

To install the chart with the name `nutanix-csi`:

```console
helm repo add nutanix https://nutanix.github.io/helm/

helm install --name nutanix-csi nutanix/nutanix-csi-storage
```

## Uninstalling the Chart

To uninstall/delete the `nutanix-csi` deployment:

```console
$ helm delete nutanix-csi
```

## Configuration

The following table lists the configurable parameters of the Nutanix-CSI chart and their default values.

|            Parameter         |                Description             |             Default            |
|------------------------------|----------------------------------------|--------------------------------|
| `prismEndPoint` | Cluster Virtual IP Address |`10.0.0.1`|
| `dataServiceEndPoint`| Prism data service IP |`10.0.0.2`|
| `username`| name used for the admin role |`admin`|
| `password`| password for the admin role |`nutanix/4u`|
| `storageContainer`| Nutanix storage container name     | `default`|
| `fsType`| type of file system you are using (ext4, xfs)  |`xfs`|
| `defaultStorageClass`| Choose your default storage class (volume, file, none) | `volume`|
| `fileHost`| NFS server IP address | `10.0.0.3`|
| `filePath`| path of the NFS share |`share`|

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
helm install --name nutanix-csi nutanix/nutanix-csi-storage --set prismEndPoint=X.X.X.X --set dataServiceEndPoint=Y.Y.Y.Y --set username=admin --set password=xxxxxxxxx --set storageContainer=container_name --set fsType=xfs --set defaultStorageClass=true
```