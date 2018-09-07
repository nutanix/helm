# Nutanix CSI plugin Helm chart

usage exemple:

```
helm repo add nutanix https://nutanix.github.io/helm/...

helm install nutanix/nutanix-csi-storage --set prismEndPoint=X.X.X.X --set dataServiceEndPoint=Y.Y.Y.Y --set username=admin --set password=xxxxxxxxx --set storageContainer=container_name --set fsType=xfs --set defaultStorageClass=true
```
