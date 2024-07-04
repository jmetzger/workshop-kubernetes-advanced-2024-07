# Metallb with helm for L2 - Installation (DigitalOcean) 

## Step 1: helm chart installi 

```
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --version 0.14.5 --namespace=metallb-system --create-namespace
```

```
# Wait for all the systems to come up
kubectl -n metallb-system get pods -o wide 
```

## Step 2: addresspool und Propagation-type (config) 

```
cd
mkdir -p manifests
cd manifests
mkdir lb
cd lb
vi 01-addresspool.yml 
```

```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  # we will use our external ip here 
  - 134.209.231.154-134.209.231.154
  # both notations are possible 
  - 157.230.113.124/32
```

```
kubectl apply -f .
```

```
vi 02-advertisement.yml
```

```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
```

```
kubectl apply -f .
```

## Schritt 4: Test do i get an external ip 

```
vi 03-deploy.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: web-nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: web-nginx
    spec:
      containers:
      - name: cont-nginx
        image: nginx
        ports:
        - containerPort: 80

```


```
vi 04-service.yml
```

```
# 02-svc.yml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  labels:
    svc: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: web-nginx
```


```
kubectl apply -f .
kubectl get pods
kubectl get svc
```

```
kubectl delete -f 03-deploy.yml 04-service.yml 
