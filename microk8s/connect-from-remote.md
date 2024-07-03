# Connect from remote 

## Step 1: Install kubectl on local machine (or jump-server)

```
# on CLIENT install kubectl 
sudo snap install kubectl --classic
```

## Step 2: configure kubectl 

```
# On MASTER -server get config 
# als root
microk8s config > /tmp/config 
```


# Download (scp config file) and store in .kube - folder  
cd ~
mkdir .kube
cd .kube  # Wichtig: config muss nachher im verzeichnis .kube liegen 
# scp kurs@master_server:/path/to/remote_config config 
# z.B. 
scp kurs@192.168.56.102:/home/kurs/remote_config config
# oder benutzer 11trainingdo
scp 11trainingdo@192.168.56.102:/home/11trainingdo/remote_config config 

#### Evtl. IP-Adresse in config zum Server aendern 

# Ultimative 1. Test auf CLIENT 
kubectl cluster-info 

# or if using kubectl or alias 
kubectl get pods 

# if you want to use a different kube config file, you can do like so 
kubectl --kubeconfig /home/myuser/.kube/myconfig

```
