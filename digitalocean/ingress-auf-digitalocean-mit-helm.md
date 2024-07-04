# Setup Ingress Controller with Digitalocean and helm 

## Basics 

  * works for all plattform with helm if no ingresscontroller ist present
  * if you have ingress - objekts and no ingresscontroller nothing works  

## Prerequisites 

  * kubectl must be set up

## Walkthrough (Setup Ingress Controller) 

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# helm show values ingress-nginx/ingress-nginx

# It will be setup with type loadbalancer - so waiting to retrieve an ip from the external loadbalancer
# This will take a little. 
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress --create-namespace --set controller.publishService.enabled=true 

# See when the external ip comes available
kubectl -n ingress get all
kubectl --namespace ingress get services -o wide -w nginx-ingress-ingress-nginx-controller

# Output  
NAME                                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE     SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.245.78.34   157.245.20.222   80:31588/TCP,443:30704/TCP   4m39s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx

# Now setup wildcard - domain for training purpose 
# inwx.com
*.lab1.t3isp.de A 157.245.20.222 


```

