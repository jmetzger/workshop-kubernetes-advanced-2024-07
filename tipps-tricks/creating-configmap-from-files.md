# Create configmap from files 

```
# Creates a users.yaml configmap-manifest 
kubectl create cm users --from-file=users.json --dry-run=client -o yaml > users.yaml
```
