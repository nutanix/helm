# Nutanix Database Service Operator for Kubernetes
The NDB operator automates and simplifies database administration, provisioning, and life-cycle management of NDB on Kubernetes.

NDB operator supports these functionalities:
1. Provisioning and deprovisioning a single instance postgres, mssql, sql server, and mongodb database with or without time machine.
2. Cloning support for the above database engines
3. Creation of a service for the applications to consume the database within Kubernetes.
---

## Pre-requisites
1. [Install](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-NDB-User-Guide-v2_5:top-installation-c.html) NDB 2.5.
2. [Install](https://helm.sh/docs/intro/install/) Helm v3.0.0.
3. [Install](https://kubernetes.io/docs/setup/) a Kubernetes cluster.
4. [Install](https://cert-manager.io/docs/installation/#getting-started) cert-manager. Ensure that the cert-manager resouces are up and running successfully before installing the NDB operator.

## Installation and Running on the cluster
Deploy the operator on the cluster:
```sh
helm repo add nutanix https://nutanix.github.io/helm/

helm install ndb-operator nutanix/ndb-operator -n ndb-operator --create-namespace
```
## Usage
###  Create secrets to be used by the NDBServer and Database resources using the manifest:

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

Create the secrets:

```
kubectl apply -f <path/to/secrets-manifest.yaml>
```

###  Create the NDBServer resource. The manifest for NDBServer is described as follows:

```yaml
apiVersion: ndb.nutanix.com/v1alpha1
kind: NDBServer
metadata:
  labels:
    app.kubernetes.io/name: ndbserver
    app.kubernetes.io/instance: ndbserver
    app.kubernetes.io/part-of: ndb-operator
    app.kubernetes.io/managed-by: kustomize
    app.kubernetes.io/created-by: ndb-operator
  name: ndb
spec:
    # Name of the secret that holds the credentials for NDB: username, password and ca_certificate created earlier
    credentialSecret: ndb-secret-name
    # NDB Server's API URL
    server: https://[NDB IP]:8443/era/v0.9
    # Set to true to skip SSL certificate validation, should be false if ca_certificate is provided in the credential secret.
    skipCertificateVerification: true

```
Create the NDBServer resource using:
```sh
kubectl apply -f <path/to/NDBServer-manifest.yaml>
```

### Create a Database Resource. A database can either be provisioned or cloned on NDB based on the inputs specified in the database manifest.

#### Provisioning manifest
```yaml
apiVersion: ndb.nutanix.com/v1alpha1
kind: Database
metadata:
  # This name that will be used within the kubernetes cluster
  name: db
spec:
  # Name of the NDBServer resource created earlier
  ndbRef: ndb
  isClone: false
  # Database instance specific details (that is to be provisioned)
  databaseInstance:
    # Cluster id of the cluster where the Database has to be provisioned
    # Can be fetched from the GET /clusters endpoint
    clusterId: "Nutanix Cluster Id"
    # The database instance name on NDB
    name: "Database-Instance-Name"
    # The description of the database instance
    description: Database Description
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
    additionalArguments:                # Optional block, can specify additional arguments that are unique to database engines.
      listener_port: "8080"

```

#### Cloning manifest
```yaml
apiVersion: ndb.nutanix.com/v1alpha1
kind: Database
metadata:
  # This name that will be used within the kubernetes cluster
  name: db
spec:
  # Name of the NDBServer resource created earlier
  ndbRef: ndb
  isClone: true
  # Clone specific details (that is to be provisioned)
  clone:
    # Type of the database to be cloned
    type: postgres
    # The clone instance name on NDB
    name: "Clone-Instance-Name"
    # The description of the clone instance
    description: Database Description
    # Cluster id of the cluster where the Database has to be provisioned
    # Can be fetched from the GET /clusters endpoint
    clusterId: "Nutanix Cluster Id"
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
    # Name of the secret with the
    # data: password, ssh_public_key
    credentialSecret: clone-instance-secret-name
    timezone: "UTC"
    # ID of the database to clone from, can be fetched from NDB REST API Explorer
    sourceDatabaseId: source-database-id
    # ID of the snapshot to clone from, can be fetched from NDB REST API Explorer
    snapshotId: snapshot-id
    additionalArguments:                # Optional block, can specify additional arguments that are unique to database engines.
      expireInDays: 3

```

Create the Database resource:
```sh
kubectl apply -f <path/to/database-manifest.yaml>
```

### Additional Arguments for Databases
Below are the various optional addtionalArguments you can specify along with examples of their corresponding values. Arguments that have defaults will be indicated.

Provisioning Additional Arguments: 
```yaml
# PostGres
additionalArguments:
  listener_port: "1111"                            # Default: "5432"

# MySQL
additionalArguments:
  listener_port: "1111"                            # Default: "3306" 

# MongoDB
additionalArguments:
  listener_port: "1111"                            # Default: "27017"
  log_size: "150"                                  # Default: "100"
  journal_size: "150"                              # Default: "100"

# MSSQL
additionalArguments:
  sql_user_name: "mazin"                           # Defualt: "sa".
  authentication_mode: "mixed"                     # Default: "windows". Options are "windows" or "mixed". Must specify sql_user.
  server_collation: "<server-collation>"           # Default: "SQL_Latin1_General_CP1_CI_AS".
  database_collation:  "<server-collation>"        # Default: "SQL_Latin1_General_CP1_CI_AS".
  dbParameterProfileIdInstance: "<id-instance>"    # Default: Fetched from profile.
  vm_dbserver_admin_password: "<admin-password>"   # Default: Fetched from database secret.
  sql_user_password:         "<sq-user-password>"  # NO Default. Must specify authentication_mode as "mixed".
  windows_domain_profile_id: <domain-profile-id>   # NO Default. Must specify vm_db_server_user.
  vm_db_server_user: <vm-db-server-use>            # NO Default. Must specify windows_domain_profile_id.
  vm_win_license_key: <licenseKey>                 # NO Default.
```

Cloning Additional Arguments: 
```yaml
MSSQL:
  windows_domain_profile_id   
  era_worker_service_user      
  sql_service_startup_account  
  vm_win_license_key           
  target_mountpoints_location  
  expireInDays                 
  expiryDateTimezone           
  deleteDatabase               
  refreshInDays                
  refreshTime                  
  refreshDateTimezone          

MongoDB:
  expireInDays                 
  expiryDateTimezone           
  deleteDatabase               
  refreshInDays                
  refreshTime                  
  refreshDateTimezone    

Postgres:
  expireInDays                 
  expiryDateTimezone           
  deleteDatabase               
  refreshInDays                
  refreshTime                  
  refreshDateTimezone  

MySQL:
  expireInDays                 
  expiryDateTimezone           
  deleteDatabase               
  refreshInDays                
  refreshTime                  
  refreshDateTimezone  
```


### Deleting the Database resource
To deregister the database and delete the VM run:
```sh
kubectl delete -f <path/to/database-manifest.yaml>
```

### Deleting the NDBServer resource
To deregister the database and delete the VM run:
```sh
kubectl delete -f <path/to/NDBServer-manifest.yaml>
```

---

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
| `imagePullSecrets`    | ImagePullSecrets list                                         | `[]`                                                   |
| `fullnameOverride`    | To override the full name of the operator chart               | `""`                                                   |
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
