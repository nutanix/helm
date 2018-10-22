# Nutanix Helm Charts repository

Adding the chart repository:

```code
helm repo add nutanix https://nutanix.github.io/helm/
```

# Adding Nutanix repository to Rancher 2.X


- Go to `Global -> Catalogs`
- Click on `Add Catalog`
- Fill in the fields
  - Name: `nutanix`
  - Catalog URL: `https://nutanix.github.io/helm/`
