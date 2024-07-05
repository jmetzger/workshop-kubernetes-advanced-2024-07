# configmap mounted as files in pod 

## Walkthrough 

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
