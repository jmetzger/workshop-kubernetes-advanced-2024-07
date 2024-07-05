# Example of sidecar 

## Walkthrough 

```
cd
mkdir -p manifests
cd manifests
mkdir -p sidecar
cd sidecar
```

```
nano 01-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example 
spec:
  securityContext:
    runAsUser: 0
    runAsGroup: 0
  containers:
  - name: splunk-uf
    image: splunk/universalforwarder:latest
    env:
    - name: SPLUNK_START_ARGS
      value: --accept-license
    - name: SPLUNK_USER
      value: root
    - name: SPLUNK_GROUP
      value: root
    - name: SPLUNK_PASSWORD
      value: helloworld
    - name: SPLUNK_CMD
      value: add monitor /var/log/
    - name: SPLUNK_STANDALONE_URL
      value: splunk.company.internal
    volumeMounts:
    - name: shared-data
      mountPath: /var/log
  - name: my-nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /var/log/nginx/
  volumes:
  - name: shared-data
    emptyDir: {}
```

```
kubectl apply -f .
kubectl get pods sidecar-example
kubectl decribe pods sidecar-example
# exec into my-nginx
kubectl exec -it sidecar-example -c my-nginx -- sh

``` 
