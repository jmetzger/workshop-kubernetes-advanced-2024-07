# Verbindung zu pod testen 

## Situation 

```
Managed Cluster und ich kann nicht auf einzelne Nodes per ssh zugreifen
Managed Cluster and i am not able to access nodes per ssh.
```

## Behelf: Eigenen Pod starten mit busybox // Bring your own pod 

```
# laengere Version / longer version 
kubectl run podtest --rm -ti --image busybox -- /bin/sh
```

```
# kuerzere Version 
kubectl run podtest --rm -ti --image busybox 
```

## Example test connection 

```
# wget (to copy)
wget -O - http://10.244.0.99
```

```
# -O -> Output (grosses O (buchstabe)) 
kubectl run podtest --rm -ti --image busybox -- /bin/sh
/ # wget -O - http://10.244.0.99
/ # exit 
```
