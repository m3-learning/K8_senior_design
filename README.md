# K8_senior_design
# Setting up NFS Server and Client on Kubernetes Cluster

## Introduction

Network File System (NFS) is a distributed file system protocol that allows a user on a client computer to access files over a network in a manner similar to how local storage is accessed. NFS, like many other protocols, builds on the Open Network Computing Remote Procedure Call (ONC RPC) system. In Kubernetes, NFS can be used to create persistent volumes that can be shared among multiple nodes in the cluster.

## Prerequisites

Before you begin, make sure you have the following:

- A Kubernetes cluster with one master and at least two worker nodes.
- `kubectl` installed on the master node.
- `helm` installed on the master node.

This guide explains how to set up an NFS Server on the master node  and configure NFS clients on Worker Nodes in a Kubernetes cluster. Additionally, it covers deploying an NFS client provisioner, creating persistent volumes and persistent volume claims, and testing the deployment.

## Step 1: Set up NFS Server on the Master Node

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
sudo mkdir /opt/dynamic-storage
sudo chown -R nobody:nogroup /opt/dynamic-storage
sudo chmod 2770 /opt/dynamic-storage
echo "/opt/dynamic-storage MASTER_NODE_IP(rw,sync,no_subtree_check) WORKER_NODE_IP(rw,sync,no_subtree_check) WORKER_NODE_IP(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

## Step 2: Install NFS Client on the Worker Nodes 

```bash 
sudo apt update
sudo apt install nfs-common -y
```

## Step 3: Install Helm on the Master Node 

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Step 4: Deploy NFS Client Provisioner on the Master Node 

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
sudo chmod 644 /users/zrm33/kube/config
helm install -n nfs-provisioning --create-namespace nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=MASTER_NODE_IP --set nfs.path=/opt/dynamic-storage
```

## Step 5: Create Persistent Volume and Persistent Volume Claim on the Master Node 
```bash
mkdir ~/kube-yaml
cd ~/kube-yaml
vi pv.yaml
```

## pv.yaml:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

## pvc.yaml:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl describe pod -n nfs-provisioning nfs-subdir-external-provisioner-67597796c5-b2vtx
```

## Step 6: Mount NFS on Worker Nodes 
```bash
sudo mkdir /mnt/test-nfs
sudo mount -t nfs MASTER_NODE_IP:/opt/dynamic-storage /mnt/test-nfs
ls /mnt/test-nfs
```

## Step 7: Test Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: test-container
        image: nginx
        volumeMounts:
        - name: test-storage
          mountPath: "/usr/share/nginx/html"
      volumes:
      - name: test-storage
        persistentVolumeClaim:
          claimName: my-pvc
```

```bash
kubectl apply -f test-deployment.yaml.
```

## Troubleshooting
If you encounter any issues during the setup process, here are a few common problems and their solutions:

Issue: NFS server not starting. Solution: Check the NFS server status using systemctl status nfs-kernel-server. If it’s not active, try restarting it with sudo systemctl restart nfs-kernel-server.

Issue: Permission denied when trying to access the NFS share from a client node. Solution: Make sure the shared directory’s permissions are correctly set on the NFS server. The directory should be owned by nobody:nogroup and have the permissions 2770

Important: Remember to replace `MASTER_NODE_IP` and `WORKER_NODE_IP` with the actual IP addresses when you're implementing these steps. 

## Conclusion
By following this guide, you’ve set up an NFS server on your Kubernetes master node and configured the worker nodes as NFS clients. You’ve also deployed an NFS client provisioner and created persistent volumes and persistent volume claims. This setup allows your Kubernetes pods to share storage, enabling more complex applications to run on your cluster. 
