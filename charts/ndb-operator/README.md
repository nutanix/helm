# Nutanix Database Service Operator for Kubernetes
The NDB operator brings automated and simplified database administration, provisioning, and life-cycle management to Kubernetes.


---

## Pre-requisites
1. NDB [installation](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-NDB-User-Guide-v2_5:Nutanix-NDB-User-Guide-v2_5).
2. A Kubernetes cluster to run against, which should have network connectivity to the NDB installation. You can use [KIND](https://sigs.k8s.io/kind) to get a local cluster for testing, or run against a remote cluster.
**Note:** The operator will automatically use the current context in your kubeconfig file (i.e. whatever cluster `kubectl cluster-info` shows).
3. The operator-sdk installed.
4. A clone of the source code ([this](https://github.com/nutanix-cloud-native/ndb-operator) repository).
5. Cert-manager. Please follow the instructions [here](https://cert-manager.io/docs/installation/), and wait for the resources created by the cert-manager installation to be ready before deploying/installing the NDB Operator.

## Installation and Running on the cluster
Deploy the operator on the cluster:
```sh
helm repo add nutanix https://nutanix.github.io/helm/

helm install ndb-operator nutanix/ndb-operator -n ndb-operator --create-namespace
```
## Using the Operator

1. Create the secrets that are to be used by the custom resource:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ndb-secret-name
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
  name: db-instance-secret-name
type: Opaque
stringData:
  password: password-for-the-database-instance
  ssh_public_key: SSH-PUBLIC-KEY

```
2. To create instances of custom resources (provision databases), edit the custom resource file with the NDB installation and database instance details and run:
```sh
kubectl apply -f CR_FILE.yaml
```
3. To delete instances of custom resources (deprovision databases) run:
```sh
kubectl delete -f CR_FILE.yaml
```
The CR is described as follows:
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
    credentialSecret: ndb-secret-name
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
      - database_one
      - database_two
      - database_three
    # Credentials secret name for NDB installation
    # data: password, ssh_public_key
    credentialSecret: db-instance-secret-name
    size: 10
    timezone: "UTC"
    type: postgres

    # You can specify any (or none) of these types of profiles: compute, software, network, dbParam
    # If not specified, the corresponding Out-of-Box (OOB) profile will be used wherever applicable
    # Name is case-sensitive. ID is the UUID of the profile. Profile should be in the "READY" state
    # "id" & "name" are optional. If none provided, OOB may be resolved to any profile of that type
    profiles:
      compute:
        id: ""
        name: ""
      # A Software profile is a mandatory input for closed-source engines: SQL Server & Oracle
      software:
        name: ""
        id: ""
      network:
        id: ""
        name: ""
      dbParam:
        name: ""
        id: ""
      # Only applicable for MSSQL databases
      dbParamInstance:
        name: ""
        id: ""
    timeMachine:                        # Optional block, if removed the SLA defaults to NONE
      sla : "NAME OF THE SLA"
      dailySnapshotTime:   "12:34:56"   # Time for daily snapshot in hh:mm:ss format
      snapshotsPerDay:     4            # Number of snapshots per day
      logCatchUpFrequency: 90           # Frequency (in minutes)
      weeklySnapshotDay:   "WEDNESDAY"  # Day of the week for weekly snapshot
      monthlySnapshotDay:  24           # Day of the month for monthly snapshot
      quarterlySnapshotMonth: "Jan"     # Start month of the quarterly snapshot

```
## Uninstalling the Chart
To uninstall/delete the operator deployment/chart:
```console
helm uninstall ndb-operator -n ndb-operator
```
---
## Configuration

The following table lists the configurable parameters of the NDB operator chart and their default values.

| Parameter             | Description                                                   | Default                                                |
|-----------------------|---------------------------------------------------------------|--------------------------------------------------------|
| `replicaCount`        | Number of replicas of the NDB Operator controller pods        | `1`                                                    |
| `image.repository`    | Image for NDB Operator controller                             | `ghcr.io/nutanix-cloud-native/ndb-operator/controller` |
| `image.pullPolicy`    | Image pullPolicy                                              | `IfNotPresent`                                         |
| `image.tag`           | Image tag                                                     | `v0.0.6, defaults to Chart.appVersion if removed`      |
| `imagePullSecrets`    | ImagePullSecrets list                                         | `[]`                                                   |
| `nameOverride`        | To override the name of the operator chart                    | `""`                                                   |
| `fullnameOverride`    | To override the full name of the operator chart               | `""`                                                   |
| `serviceAccount.name` | Name of the service account that will be used by the operator | `ndb-operator-service-account`                         |
| `podAnnotations`      | Add annotation to NDB Operator controller pods                | `kubectl.kubernetes.io/default-container: manager`     |
| `podSecurityContext`  | Security context for the pod(s) running the operator          | `runAsNonRoot: true`                                   |
| `securityContext`     | Security context for the container running the controller     | `allowPrivilegeEscalation: false`                      |
| `resources`           | Configure resources for Cloud Provider Pod                    | `refer to values.yaml`                                 |
| `nodeSelector`        | Configure nodeSelector for Cloud Provider Pod                 | `refer to values.yaml`                                 |
| `tolerations`         | Configure tolerations for Cloud Provider Pod                  | `refer to values.yaml`                                 |
| `affinity`            | Configure affinity for Cloud Provider Pod                     | `refer to values.yaml`                                 |


### Configuration examples:

Install the operator in the `ndb-operator` namespace (add the `--create-namespace` flag if the namespace does not exist): 

```console
helm install ndb-operator nutanix/ndb-operator -n ndb-operator 
```

Individual configurations can be set by using `--set key=value[,key=value]` like:
```console
helm install ndb-operator nutanix/ndb-operator  --set replicaCount=2 
```
In the above command `replicaCount` refers to one of the variables defined in the values.yaml file. 

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

Copyright 2022-2023 Nutanix, Inc.

The project is released under version 2.0 of the [Apache license](http://www.apache.org/licenses/LICENSE-2.0).
