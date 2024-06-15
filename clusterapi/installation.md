# Installation mit Kubernetes Cluster API (Digitalocean) 

## Voraussetzungen:

  * Cluster erstellt mit kubeadm 
  * Zugriff zu Cluster auf ./kube/config eingerichtet.
  * clusterctl installieren
  * kubectl installieren 

## Schritt 1: Cluster erstellen mit kubeadm

  * Wir verwenden dafür ein paar selbsterstellte Scripte

```
# Ein Cluster erstellen 
cd multi-kubeadmin
./create-multi.sh 1
# Alternativ: Gleich mehrere Cluster für das Training erstellen
# ./create-multi.sh 2
```

## Schritt 2: kubectl runterladen und installieren 

```
# Variante 1:
# Systemd needs to work with wsl
# https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/
sudo su - 
snap install --classic kubectl 
```

```
# Variante 2: Download and install binary (untested)
sudo su -
# https://kubernetes.io/de/docs/tasks/tools/install-kubectl/#installation-der-kubectl-anwendung-mit-curl
curl -LO https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x ./kubectl
mv kubectl /usr/local/bin
```

## Schritt 3: kubeconfig einrichten und Verbindung prüfen 

```
# aus dem Onlinesystem unter /etc/kubernetes/admin.conf den Inhalt kopieren
# nach lokal (unpriviligierter Nutzer)
# in
cd
cd .kube
vi config
```

```
kubectl cluster-info
```

## Schritt 4: clusterctl installieren 

```
sudo su -
cd /usr/src 
curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.4.2/clusterctl-linux-amd64 -o clusterctl
sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl
clusterctl version 
```

## Schritt 5: Clusterctl initialisieren (mit dem richtigen provider)

```
# Feature Gate, das für die cluster-api gebraucht wird 
export CLUSTER_TOPOLOGY=true
export DIGITALOCEAN_ACCESS_TOKEN=<your-access-token>
export DO_B64ENCODED_CREDENTIALS="$(echo -n "${DIGITALOCEAN_ACCESS_TOKEN}" | base64 | tr -d '\n')"

# Initialize the management cluster
clusterctl init --infrastructure digitalocean
```

## Schritt 6: install doctl (used to interact with digitalocean) 

```
cd 
wget https://github.com/digitalocean/doctl/releases/download/v1.107.0/doctl-1.107.0-linux-amd64.tar.gz
tar xf ~/doctl-1.107.0-linux-amd64.tar.gz
sudo mv ~/doctl /usr/local/bin
sudo chmod +x /usr/local/bin/doctl 
```

## Schritt 7: install image builder for creating image for digitalocean 

```
# if not already done before 
export DIGITALOCEAN_ACCESS_TOKEN=<your-access-token>
# as unprivileged user 
git clone https://github.com/kubernetes-sigs/image-builder.git
cd image-builder/images/capi
# install dependencies 
make deps-do
# show possible builds
make help
# Size of machine will always be 1gb and 1vcpu created in NYC1
make build-do-ubuntu-2404
```

  * ACHTUNG: Das dauert eine ganze Weile das bauen

![image](https://github.com/jmetzger/training-kubernetes-advanced/assets/1933318/61cfdf53-d97e-4b0f-9b8e-1b2d0445b857)

![image](https://github.com/jmetzger/training-kubernetes-advanced/assets/1933318/b159c6fe-35c2-443f-a265-8cdf4ac2ef55)

## Schritt 8: Which kubernetes cluster version is it ?

```
# you need to use exactly the same version for creating your workload cluster
Creating snapshot: Cluster API Kubernetes v1.28.9 on Ubuntu 24.04
```

## Schritt 9: Allow Image to be use in Frankfurt Datacenter (FRA1) 

```
-> Add to Region FRA1 -> under Manage -> Backups&Snaphots -> Snapshots 
Please do this through the web-interface of DigitalOcean 
# IF YOU DO NOT DO THIS... Droplets cannot be created because they are in NYC1
```


## References:

  * https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl
  * https://cluster-api.sigs.k8s.io/user/quick-start


## Installation of Kubernetes Cluster API

## Prerequisites 

  * You need to have a Kubernetes Cluster running (this will be the management cluster) 
    * Within that you will you have cluster api 
    * This could be something like kind, rancherdesktop.io
    * And of course also a cluster on premise 

## Step 1: Create Management Cluster 

### Step 1a: Install clusterctl 

```
# Install rancherdesktop.io 
# You are able to use it on windows (that's what we do now, and install 

# Install clusterctl in wsl -> Ubuntu 
sudo su -
cd /usr/src 
curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.4.2/clusterctl-linux-amd64 -o clusterctl
sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl
clusterctl version 
```

  * Reference gist: https://gist.github.com/vfarcic/d8113b6f149583e1cf1614d76f2a4182
  * https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl

### Step 1b: Set env variables for digitalocean 

```
export DIGITALOCEAN_ACCESS_TOKEN=[...] # Replace with your token here 

```


### Step 1c: Create kubernetes snapshot to be used for Kubernetes Control Plane and workers 

``` 
# can be done as unprivileged user !!! 
export PATH=$PATH:~/.local/bin
sudo apt update
apt install -y jq zip 
sudo git clone https://github.com/kubernetes-sigs/image-builder
cd image-builder/images/capi
cat Makefile
# Size of machine will always be 1gb and 1vcpu created in NYC1 
make build-do-ubuntu-2004
```

### Step 1d: Add Snapshot to Region FRA1

```
-> Add to Region FRA1 -> under Manage -> Images -> Snapshots 
Please do this through the web-interface of DigitalOcean 
# IF YOU DO NOT DO THIS... Droplets cannot be created because they are in NYC1
```

### Step 1e: Install doctl (optional)

```
# works in most cases on wsl, but only if snap is working properly 
# snap install doctl 
# if not do -> this 

cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.94.0/doctl-1.94.0-linux-amd64.tar.gz
tar xf ~/doctl-1.94.0-linux-amd64.tar.gz
sudo mv ~/doctl /usr/local/bin

# now authenticate 
doctl auth init --access-token ${DIGITALOCEAN_ACCESS_TOKEN}

```

### Step 1f: Set env for to create worker cluster with controlplane and workers 

```
# control the datacenter - default nyc1 
export DO_REGION=fra1
# control size of machines
# default 1vcpu-1gb 
export DO_CONTROL_PLANE_MACHINE_TYPE=s-2vcpu-2gb
export DO_NODE_MACHINE_TYPE=s-2vcpu-2gb
# needed to set up the api provider 
export DO_B64ENCODED_CREDENTIALS="$(\
    echo -n "$DIGITALOCEAN_ACCESS_TOKEN" \
    | base64 \
    | tr -d '\n')"
    
# get the snapshot id / get the right id 
doctl compute image list-user
# e.g. 
# 132627725

export DO_CONTROL_PLANE_MACHINE_IMAGE=132627725
export DO_NODE_MACHINE_IMAGE=132627725

```

### Step 1g: Setup cluster and api-provider 

```
## In our case it sets up the management cluster on rancher 
## to be used for kubernetes 
cd ../../../

clusterctl init \
    --infrastructure digitalocean

```

### Step 1h: Generate the yaml scripts for both control plane and workers 

```
# it looks there will be a fingerprint to be used, which chooses the ssh-key to be used
# to connect to the machines
# look for all the ssh-key like so:
doctl compute ssh-key list 

# So we choose one from the list 
export DO_SSH_KEY_FINGERPRINT=[...]

# Check the variables 
# Show use the necessary env-variables.
clusterctl generate cluster devops-toolkit \
    --infrastructure digitalocean \
    --target-namespace infra \
    --kubernetes-version v1.24.11 \
    --control-plane-machine-count 3 \
    --worker-machine-count 3 \
    --list-variables 


# Kuberentes must be the same version as you created the snapshots for do
# to be used for digitalocean -> creating a cluster there
clusterctl generate cluster devops-toolkit \
    --infrastructure digitalocean \
    --target-namespace infra \
    --kubernetes-version v1.24.11 \
    --control-plane-machine-count 3 \
    --worker-machine-count 3 \
    | tee cluster.yaml
    
kubectl create namespace infra

kubectl apply --filename cluster.yaml
```

### Step 1i: Wait till the control plane is initialized + install calico 

```
kubectl get kubeadmcontrolplane

# When initialized get kubeconfig and install calicao 
clusterctl --namespace infra2 \
    get kubeconfig devops-toolkit \
    | tee kubeconfig.yaml


kubectl --kubeconfig kubeconfig.yaml get ns
# you will see control plane is not ready because of network missing
kubectl --kubeconfig kubeconfig.yaml get nodes

kubectl --kubeconfig kubeconfig.yaml apply -f https://docs.projectcalico.org/v3.25/manifests/calico.yaml

```

### Step 1j: READY it is (says Yoda) 

```
# Wait a while, now you will see, the nodes are ready
kubectl --kubeconfig kubeconfig.yaml get nodes 
``` 
