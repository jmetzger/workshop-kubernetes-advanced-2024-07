# configmap mounted as files in pod 

## Walkthrough 

```
cd
mkdir -p manifests
cd manifests
mkdir cm-files
cd cm-files
```

```
nano 01-cm.yml
```


```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html-configmap
data:
  index.html: |
    <html>
    <h1>Namaskar Mitrano</h1>
    <body>
    This is Nginx Server
    </body>
    </html>
```

```
nano 02-pod.yml
```


```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
      - name: nginx-index-file
        mountPath: /usr/share/nginx/html/
  volumes:
  - name: nginx-index-file
    configMap:
      name: index-html-configmap

```


```
kubectl apply -f .
kubectl get pods -o wide 
kubectl run -it --rm podtest --image=busybox  
```

```
# in pod
wget -O - <ip-of-pod>
```

