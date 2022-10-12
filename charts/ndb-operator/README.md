# Nutanix Database Service Operator for Kubernetes
The NDB operator brings automated and simplified database administration, provisioning, and life-cycle management to Kubernetes.

---

## Pre-requisites
1. NDB [installation](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Era-User-Guide-v2_4:top-era-installation-c.html).
2. A Kubernetes cluster.
3. Helm installed.

## Installation and Running on the cluster
Deploy the operator on the cluster:
```sh
helm repo add nutanix https://nutanix.github.io/helm/

helm install ndb-operator nutanix/ndb-operator --version 0.0.2
```
## Using the Operator

1. Create the secrets that are to be used by the custom resource:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: your-ndb-secret
type: Opaque
stringData:
  username: username-for-ndb-server
  password: password-for-ndb-server
  ca_certificate: |
    -----BEGIN CERTIFICATE-----
    CA CERTIFICATE (ca_certificate is optional)
    -----END CERTIFICATE-----
---
apiVersion: v1
kind: Secret
metadata:
  name: your-db-secret
type: Opaque
stringData:
  password: password-for-the-database-instance
  ssh_public_key: SSH-PUBLIC-KEY

```
2. To create instances of custom resources (provision databases), edit the crd file with the NDB installation and database instance details and run:
```sh
kubectl apply -f CRD_FILE.yaml
```
3. To delete instances of custom resources (deprovision databases) run:
```sh
kubectl delete -f CRD_FILE.yaml
```
The CRD is described as follows:
```yaml
apiVersion: ndb.nutanix.com/v1alpha1
kind: Database
metadata:
  # This name that will be used within the kubernetes cluster
  name: db
spec:
  # NDB server specific details
  ndb:
    # Cluster id of the cluster where the Database has to be provisioned
    # Can be fetched from the GET /clusters endpoint
    clusterId: "Nutanix Cluster Id" 
    # Credentials secret name for NDB installation
    # data: username, password, 
    # stringData: ca_certificate
    credentialSecret: your-ndb-secret
    # The NDB Server
    server: https://[NDB IP]:8443/era/v0.9
    # Set to true to skip SSL verification, default: false.
    skipCertificateVerification: true
  # Database instance specific details (that is to be provisioned)
  databaseInstance:
    # The database instance name on NDB
    databaseInstanceName: "Database-Instance-Name"
    # Names of the databases on that instance
    databaseNames:
      - alpha
      - beta
    # Credentials secret name for NDB installation
    # data: password, ssh_public_key
    credentialSecret: your-db-secret
    size: 10
    timezone: "UTC"
    type: postgres
```
## Uninstalling the Chart
To uninstall/delete the operator deployment/chart:
```console
helm uninstall [RELEASE_NAME]
```
---
## Configuration

The following table lists the configurable parameters of the NDB operator chart and their default values.

| Parameter                   | Description                                                      | Default                                                          |
|-----------------------------|------------------------------------------------------------------|------------------------------------------------------------------|
| `replicaCount`              | Number of replicas of the NDB Operator controller pods           | `1`                                                              |
| `image.repository`          | Image for NDB Operator controller                                | `ghcr.io/nutanix-cloud-native/ndb-operator/controller`           |
| `image.pullPolicy`          | Image pullPolicy                                                 | `IfNotPresent`                                                   |
| `image.tag`                 | Image tag                                                        | `v0.0.2, defaults to Chart.appVersion if removed`                |
| `imagePullSecrets`          | ImagePullSecrets list                                            | `[]`                                                             |
| `nameOverride`              | To override the name of the operator chart                       | `""`                                                             |
| `fullnameOverride`          | To override the full name of the operator chart                  | `""`                                                             |
| `serviceAccount.name`       | Name of the service account that will be used by the operator    | `ndb-operator-service-account`                                   |
| `podAnnotations`            | Add annotation to NDB Operator controller pods                   | `kubectl.kubernetes.io/default-container: manager`               |
| `podSecurityContext`        | Security context for the pod(s) running the operator             | `runAsNonRoot: true`                                             |
| `securityContext`           | Security context for the container running the controller        | `allowPrivilegeEscalation: false`                                |
| `resources`                 | Configure resources for Cloud Provider Pod                       | `refer to values.yaml`                                           |
| `nodeSelector`              | Configure nodeSelector for Cloud Provider Pod                    | `refer to values.yaml`                                           |
| `tolerations`               | Configure tolerations for Cloud Provider Pod                     | `refer to values.yaml`                                           |
| `affinity`                  | Configure affinity for Cloud Provider Pod                        | `refer to values.yaml`                                           |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install` or provide a file with `-f value.yaml`.

### Configuration examples:

Install the operator in the `ndb-operator-ns` namespace (add the `--create-namespace` flag if the namespace does not exist): 

```console
helm install ndb-operator nutanix/ndb-operator -n ndb-operator-ns 
```

Individual configurations can be set by using `--set key=value[,key=value]` like:
```console
helm install ndb-operator nutanix/ndb-operator  --set fullnameOverride=my-operator --set replicaCount=2 
```
In the above command  `fullnameOverride`, `replicaCount` refers to some of the variables defined in the values.yaml file. 

All the options can also be specified in a value.yaml file:

```console
helm install ndb-operator nutanix/ndb-operator -f value.yaml
```
---

## How it works

This project aims to follow the Kubernetes [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

It uses [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/) 
which provides a reconcile function responsible for synchronizing resources until the desired state is reached on the cluster.

A custom resource of the kind Database is created by the reconciler, followed by a Service and an Endpoint that maps to the IP address of the database instance provisioned. Application pods/deployments can use this service to interact with the databases provisioned on NDB through the native Kubernetes service. 

Pods can specify an initContainer to wait for the service (and hence the database instance) to get created before they start up.
```yaml
  initContainers:
  - name: init-db
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup <<Database CR Name>>-svc.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for database service; sleep 2; done"]
```

## Contributing
See the [contributing docs](https://github.com/nutanix-cloud-native/ndb-operator/blob/main/CONTRIBUTING.md).

## Support
### Community Plus

This code is developed in the open with input from the community through issues and PRs. A Nutanix engineering team serves as the maintainer. Documentation is available in the project repository.

Issues and enhancement requests can be submitted in the [Issues tab of this repository](https://github.com/nutanix-cloud-native/ndb-operator/issues). Please search for and review the existing open issues before submitting a new issue.

## License

Copyright 2021-2022 Nutanix, Inc.

The project is released under version 2.0 of the [Apache license](http://www.apache.org/licenses/LICENSE-2.0).
