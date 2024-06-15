# Installation mit Kubernetes Cluster API (Digitalocean) 

## Komponenten 

  1. 1x Management-Cluster
  1. beliebig viele Workload Cluster, die vom Management Cluster verwaltet werden

## Voraussetzungen (für Management - Cluster) 

  * Cluster erstellt mit kubeadm (für Management - Cluster) 
  * Zugriff zu Cluster auf ./kube/config eingerichtet.
  * clusterctl installieren
  * kubectl installieren 

## Phase 1: Kubernetes-Cluster erstellen und zu Management-Cluster upgraden 

## Schritt 1.1: Cluster erstellen mit kubeadm

  * Wir verwenden dafür ein paar selbsterstellte Scripte

```
# Ein Cluster erstellen 
cd multi-kubeadmin
./create-multi.sh 1
# Alternativ: Gleich mehrere Cluster für das Training erstellen
# ./create-multi.sh 2
```

## Schritt 1.2: kubectl runterladen und installieren 

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

## Schritt 1.3: kubeconfig einrichten und Verbindung prüfen 

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

## Schritt 1.4: clusterctl installieren 

```
sudo su -
cd /usr/src 
curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.4.2/clusterctl-linux-amd64 -o clusterctl
sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl
clusterctl version 
```

## Schritt 1.5: Clusterctl initialisieren (mit dem richtigen provider)

```
# Feature Gate, das für die cluster-api gebraucht wird 
export CLUSTER_TOPOLOGY=true
export DIGITALOCEAN_ACCESS_TOKEN=<your-access-token>
export DO_B64ENCODED_CREDENTIALS="$(echo -n "${DIGITALOCEAN_ACCESS_TOKEN}" | base64 | tr -d '\n')"

# Initialize the management cluster
clusterctl init --infrastructure digitalocean
```

## Phase 2: Spezielles Kubernetes Images verweden 

  * Direkt mit allen Komponenten im Bauch in einer speziellen Version (z.B. 1.28.9)
  * die für Kubernetes gebraucht werden 

## Schritt 2.1: install doctl (used to interact with digitalocean) 

```
cd 
wget https://github.com/digitalocean/doctl/releases/download/v1.107.0/doctl-1.107.0-linux-amd64.tar.gz
tar xf ~/doctl-1.107.0-linux-amd64.tar.gz
sudo mv ~/doctl /usr/local/bin
sudo chmod +x /usr/local/bin/doctl 
```

## Schritt 2.2: install image builder for creating image for digitalocean 

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

## Schritt 2.3: Which kubernetes cluster version is it ?

```
# you need to use exactly the same version for creating your workload cluster
Creating snapshot: Cluster API Kubernetes v1.28.9 on Ubuntu 24.04
```

## Schritt 2.4: Allow Image to be use in Frankfurt Datacenter (FRA1) 

```
-> Add to Region FRA1 -> under Manage -> Backups&Snaphots -> Snapshots 
Please do this through the web-interface of DigitalOcean 
# IF YOU DO NOT DO THIS... Droplets cannot be created because they are in NYC1
```

## Phase 3: Workload - Cluster mit Cluster - API erstellen 

## Schritt 3.1 Umgebung zum Ausrollen von ControlNode und Worker vorbereiten  

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
```

## Schritt 3.2 Snaphot-id ausfindig machen 

```
doctl compute image list-user
```

```
158401784    Cluster API Kubernetes v1.28.9 on Ubuntu 24.04
```

## Schritt 3.3 Use this snapshot for creation of workload-cluster 

```
export DO_CONTROL_PLANE_MACHINE_IMAGE=158401784
export DO_NODE_MACHINE_IMAGE=158401784
```

## Schritt 3.4 Wir brauchen ein ssh-key 

```
# das sollte ein Schlüssel sein, für den wir bereits einen privaten Schlüssel haben
# und den öffentlichen bei digitalocean hochgeladen haben.
# Dieser wird dann für die Maschinen verwendet, die hochgezogen werden
doctl compute ssh-key list 

# wir nehmen den kubernetes key
42134500    key_training_kubernetes
```

```
# So übergeben wir diesen für das doctl - Tool
export DO_SSH_KEY_FINGERPRINT=42134500
```

## Schritt 3.5 Cluster.yaml (config) für cluster-api erstellen 

```
# Achtung, es muss die gleiche version verwendet werden für die kubernetes version, wie im image
# das mit dem Image-Builder erstellt wurde
```

```
# Check the variables 
# Show use the necessary env-variables.
clusterctl generate cluster cluster1 \
    --infrastructure digitalocean \
    --target-namespace infra \
    --kubernetes-version v1.28.9 \
    --control-plane-machine-count 1 \
    --worker-machine-count 3 \
    --list-variables 
```

```
# Now create the cluster.yaml file (config to create it)
# Kuberentes must be the same version as you created the snapshots for do
# to be used for digitalocean -> creating a cluster there
clusterctl generate cluster cluster1 \
    --infrastructure digitalocean \
    --target-namespace infra \
    --kubernetes-version v1.28.9 \
    --control-plane-machine-count 1 \
    --worker-machine-count 3 \
    | tee cluster.yaml
```

```
# Create namespace and management cluster for that
kubectl create namespace infra

# and create it 
kubectl apply --filename cluster.yaml
```

## Schritt 3.5: Wait till controlplane is ready 

```
# Achtung das dauert wieder eine ganze Weile,
# Man kann das im Backend von Digitalocean beobachten
kubectl -n infra get kubeadmcontrolplane
kubectl -n infra get domachines
```

## Schritt 3.6: Get kubeconfig 

```
# When initialized get kubeconfig 
clusterctl --namespace infra \
    get kubeconfig cluster1 \
    | tee kubeconfig.yaml
```

## Schritt 3.7. Access new cluster with kubeconfi 

```
kubectl --kubeconfig kubeconfig.yaml get ns
# Are all pods ready / of coredns not ;o) 
kubectl --kubeconfig kubeconfig.yaml -n kube-system get pods 
kubectl --kubeconfig kubeconfig.yaml get nodes
# |    | nodes are not ready, because the is no cni-provider installed yet
# v    v 
```

![image](https://github.com/jmetzger/training-kubernetes-advanced/assets/1933318/b2fa4c0c-378b-49d8-8d57-426193d7ae75)

## Schritt 3.8. Now install cni -> calico (we will use the operator)

```
kubectl --kubeconfig kubeconfig.yaml create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
# We also want the crd's
kubectl --kubeconfig kubeconfig.yaml create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml
```

## Schritt 3.9. See the rollout and findout, if everything works 

```
# Now watch if everything works smoothly
# everything should be ready 
watch kubectl --kubeconfig kubeconfig-yaml get pods -n calico-system
```

```
# + the kube-system coredns pod should also be ready now
kubectl
```


### Schritt 3.10 Installing the CCM (Cloud Controller Manager for digitalocean) 

  * Important: Before that, the calico-controller will not run, because it cannot get scheduled
  * There is a taint on the nodes
    * node.cloudprovider.kubernetes.io/uninitialized
    * There is another taint as well
  * These 2 taints will first get removed, once the CCM is installed 

```
kubectl --kubeconfig=kubeconfig.yaml apply -f https://raw.githubusercontent.com/digitalocean/digitalocean-cloud-controller-manager/v0.1.54/releases/digitalocean-cloud-controller-manager/v0.1.54.yml
```

## References:

  * https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl
  * https://cluster-api.sigs.k8s.io/user/quick-start
  * https://github.com/kubernetes-sigs/cluster-api-provider-digitalocean/blob/main/docs/getting-started.md
