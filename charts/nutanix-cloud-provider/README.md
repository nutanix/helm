# Nutanix Cloud Provider Helm chart

## Introduction

The cloud-controller-manager is a Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.

By decoupling the interoperability logic between Kubernetes and the underlying cloud infrastructure, the cloud-controller-manager component enables cloud providers to release features at a different pace compared to the main Kubernetes project.

The cloud-controller-manager is structured using a plugin mechanism that allows different cloud providers to integrate their platforms with Kubernetes.

The Nutanix Cloud Provider is a plugin that allows Nutanix AHV platform integration with Kubernetes by implementing the node controller function.

The node controller is responsible for updating [Node](https://kubernetes.io/docs/concepts/architecture/nodes/) objects when new VMs are created in your Nutanix infrastructure. The node controller obtains information about the hosts running inside your tenancy with the Nutanix Prism Central API. The node controller performs the following functions:

1. Update a Node object with the corresponding server's unique identifier obtained from the Nutanix Prism Central API.
2. Annotating and labelling the Node object with Nutanix-specific information, such as the region the node is deployed into and the nodes the VMs are running on.
3. Obtain the node's hostname and network addresses.
4. Verifying the node's health. In case a node becomes unresponsive, this controller checks with the Nutanix Prism Central API to see if the server has been deactivated / deleted / terminated. If the node has been deleted from the Nutanix infrastructure, the controller deletes the Node object from your Kubernetes cluster.



## Prerequisites

- The Kubernetes cluster needs to be deployed with cloud-provider set to `external`
- Read access account to the Prism Central instance



## Installing the Chart

To install the chart with the name `nutanix-ccm`:

```console
helm repo add nutanix https://nutanix.github.io/helm/

helm install nutanix-ccm nutanix/nutanix-cloud-provider -n <namespace of your choice>
```



## Upgrade

Upgrades can be done using the normal Helm upgrade mechanism

```
helm repo update
helm upgrade nutanix-ccm nutanix/nutanix-cloud-provider -n <namespace of your choice>
```



## Uninstalling the Chart

To uninstall/delete the `nutanix-csi` deployment:

```console
helm delete nutanix-ccm -n <namespace of your choice>
```

## Configuration

The following table lists the configurable parameters of the Nutanix-CSI chart and their default values.

| Parameter                   | Description                                                      | Default                                                          |
|-----------------------------|------------------------------------------------------------------|------------------------------------------------------------------|
| `createConfig`              | Create config for Nutanix Cloud Provider (if false use existing) | `true`                                                           |
| `configName`                | Name of the ConfigMap for Nutanix Cloud Provider config          | `nutanix-config`                                                 |
| `prismCentralEndPoint`      | Hostname or IP to connect to Prism Central instance              | `10.0.0.1`                                                       |
| `prismCentralPort`          | Port to connect to Prism Central instance                        | `9440`                                                           |
| `prismCentralInsecure`      | Allow insecure server connections to Prism Central instance      | `false`                                                          |
| `createSecret`              | Create secret for Nutanix Cloud Provider (if false use existing) | `true`                                                           |
| `secretName`                | Name of the secret for Nutanix Cloud Provider credentials        | `nutanix-creds`                                                  |
| `username`                  | Username to connect to Prism Central instance                    | `cpi`                                                            |
| `password`                  | Password to connect to Prism Central instance                    | `nutanix/4u`                                                     |
| `enableCustomLabeling`      | Add some additional custom Nutanix labels to nodes               | `false`                                                          |
| `topologyDiscovery.type`    | Define how Topology will be discovered (Prism or Categories)     | `Prism`                                                          |
| `topologyCategories.region` | Category name used to assign region topology                     | `region`                                                         |
| `topologyCategories.zone`   | Category name used to assign zone topology                       | `zone`                                                           |
| `replicas`                  | Number of instance(s) of Cloud Provider Pod                      | `1`                                                              |
| `image.repository`          | Image for Cloud Provider Pod                                     | `ghcr.io/nutanix-cloud-native/cloud-provider-nutanix/controller` |
| `image.pullPolicy`          | Image pullPolicy                                                 | `IfNotPresent`                                                   |
| `image.tag`                 | Image tag                                                        | `appVersion`                                                     |
| `imagePullSecrets`          | ImagePullSecrets list                                            | `[]`                                                             |
| `podAnnotations`            | Add annotation to Cloud Provider Pod                             | `{}`                                                             |
| `resources`                 | Configure resources for Cloud Provider Pod                       | `refer to values.yaml`                                           |
| `nodeSelector`              | Configure nodeSelector for Cloud Provider Pod                    | `refer to values.yaml`                                           |
| `tolerations`               | Configure tolerations for Cloud Provider Pod                     | `refer to values.yaml`                                           |
| `affinity`                  | Configure affinity for Cloud Provider Pod                        | `refer to values.yaml`                                           |



Specify each parameter using the `--set key=value[,key=value]` argument to `helm install` or provide a file with `-f value.yaml`.

### Configuration examples:

Install the provider in the `kube-system` namespace:

```console
helm install nutanix-ccm nutanix/nutanix-cloud-provider -n kube-system --set prismCentralEndPoint=X.X.X.X --set username=admin --set password=xxxxxxxxx --set prismCentralInsecure=true
```
In the above command  `prismCentralEndPoint`, `username`, `password`, `prismCentralInsecure` refers to the Prism Central information where the K8s cluster is deployed. 

All the options can also be specified in a value.yaml file:

```console
helm install nutanix-ccm nutanix/nutanix-cloud-provider -n kube-system -f value.yaml
```
## Contributing
See the [contributing docs](../../CONTRIBUTING.md).

## Support
### Community Plus

This code is developed in the open with input from the community through issues and PRs. A Nutanix engineering team serves as the maintainer. Documentation is available in the project repository.

Issues and enhancement requests can be submitted in the [Issues tab of this repository](../../issues). Please search for and review the existing open issues before submitting a new issue.
