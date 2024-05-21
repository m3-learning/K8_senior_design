# Deployment-Docs.md
An outline on services to deploy and setup a working HA Kubernetes environment.

## Table of Contents
1. [Requirements](#requirements)
2. [Purpose](#purpose)
3. [Kubespray](#kubespray)
4. [Install local-path on both clusters](#install-local-path-on-both-clusters)
5. [Install Metallb](#install-metallb)
6. [Install Kubernetes Dashboard](#install-kubernetes-dashboard)
7. [Install Kube-Prometheus-Stack](#install-kube-prometheus-stack)
8. [Install Gitea](#install-gitea)
9. [Install Gitea Act Runners](#install-gitea-act-runners)
10. [Install ArgoCD](#install-argocd)
11. [Install Ingress NGINX](#install-ingress-nginx)
12. [Known Issues](#known-issues)

## Requirements
The bare minimum for a Kubernetes environment is 3 per cluster. In this document, two clusters are created with the aim to provide stability, a working, and stable environment.

Production Cluster:
- 6 nodes with 8192 RAM and 4 vCPU, all configured as control plane
- 50 gb each

Test Cluster:
- 1 node with 8192 RAM and 4 vCPU as control plane
- 2 nodes with 4096 RAM and 3 vCPU as worker
- 50 gb each

## Purpose
The production cluster is completely separate from the test cluster, and while it is possible to use separate namespaces on a single cluster - this implies that all resources are shared. You can also set different prioritizations for pods, but I opted in for two separate clusters. This ensures:
1. Resources are separate between clusters
2. Enhance security for production cluster
3. If either clusters go down, the other can still perform their tasks.

The obvious drawbacks are the complexities of managing two separate clusters. However, this design allows the clusters to scale depending on needs of the organization and prioritization of resources.

Additionally, unless specified, assume all `install` sections are for the production cluster only.

## Kubespray
Ensure that Python3.10 is installed on controller for preparing installation. Create an inventory file that can be generated through Kubespray's official documentation. Configure and tune as required.

Sample Production:
```
# Inventory for production based cluster
all:
  hosts:
    node1:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd01
    node2:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd02
    node3:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd03
    node4:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd04
    node5:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd05
    node6:
      ansible_host:
      ip:
      access_ip:
      ansible_user: sd06
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
        node6:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
        node6:
    etcd:
      hosts:
        node1:
        node3:
        node5:
    k8s_cluster:
      children:
        kube_control_plane:
          node1:
          node2:
          node3:
          node4:
          node5:
          node6:
        kube_node:
          node1:
          node2:
          node3:
          node4:
          node5:
          node6:
    calico_rr:
      hosts: {}
```
Sample Test:
```
# Inventory for test cluster
all:
  hosts:
    node1:
      ansible_host:
      ip:
      access_ip:
      ansible_user: testsd01
    node2:
      ansible_host:
      ip:
      access_ip:
      ansible_user: testsd02
    node3:
      ansible_host:
      ip:
      access_ip:
      ansible_user: testsd03
  children:
    kube_control_plane:
      hosts:
        node1:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
    etcd:
      hosts:
        node1:
    k8s_cluster:
      children:
        kube_control_plane:
          node1:
        kube_node:
          node1:
          node2:
          node3:
    calico_rr:
      hosts: {}
```

Run Kubespray on both inventories:
```
ansible-playbook -i <path to inventory> -b -v --become-user=root --ask-become-pass cluster.yml
```

After running Kubespray, a kubeconfig named `config` will be generated on the control plane nodes of each cluster. You can access this in the root directory. Ensure that you have proper permissions and use your protocol of choice to get the `config` to your local computer. This is more useful for running `kubectl` in the future, and is highly recommended to do so. For specifiying the clusters to work on, use `--kubeconfig <config file name>` or manually set name to `config` in the `~/.kube` directory.

## Install local-path on both clusters
https://github.com/rancher/local-path-provisioner

Run the config map installation, then the pod deployment. The pod deployment is a DaemonSet as it specifically needs to allocate storage across its respective resources.

```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml
kubectl apply -f local-path-configmap.yml
kubectl apply -f local-path.yml
```

This should be ran on both clusters, then ensure to set the storage class is set as default. This is important because without the `default` kubernetes will not be able to dynamically provision resources (i.e. PV and PVC).

```
https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/

kubectl patch storageclass <storage-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Install Metallb
Metallb offers a solution for exposing services from an on-prem infrastructure.

```
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb -f metallb.yaml
```

## Install Kubernetes Dashboard
An optional dashboard is provided from kubernetes and can be installed: https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard?modal=install

```
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard
helm install kubernetes-dashboard k8s-dashboard/kubernetes-dashboard -f dashboard.yml
```

# Install Kube-Prometheus-Stack
This stack offers an all-in-one solution for installing Prometheus and Grafana for displaying vital information on cluster resources. The complexity of this deployment is not covered in this document, and it is highly encouraged to read their docs: https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f kube-prometheus.yml
```

# Install Gitea
Gitea is a service that provides version control solutions similar to GitLab and GitHub. The reasoning for using Gitea over GitLab is that the helm installation for GitLab is cumbersome, and GitLab in general utilizes more resources than Gitea. Passwords presented in the `values` file should be changed, and possibly using a secrets resource where it can read from the secret directly.

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install gitea bitnami/gitea -f gitea.yml
```

# Install Gitea Act Runners
A separate Helm chart is provided to this repository, and will be used for spinning up runners for the Gitea service. Please note that this is a temporary solution for installing Gitea Runner, and should be modified given that the secrets are hardcoded into the helm chart value.

Login to Gitea as the administrator and go to settings. Then, go to the runners page and copy the token for registering new act runners. Modify the `gitea-act-runner-values.yml` with the token.

```
helm install gitea-act-runner-1 gitea-act-runner-helm -f gitea-act-runner-values.yml
```

You can add more runners:

```
helm install gitea-act-runner-2 gitea-act-runner-helm -f gitea-act-runner-values.yml
```

This spins up multiple replicas of the runner that are utilized, and you should see the runners added to Gitea.

# Install ArgoCD
This section covers the deployment and installation of ArgoCD. Please note that this section relies on the two clusters for setting this up properly. On the test cluster, it is necessary to create a separate user for argocd such that you can obtain the bearer token.

```
# test cluster

# This was obtained from https://superuser.com/questions/1394619/how-to-get-the-admin-user-token-from-kubectl
k create serviceaccount argocd -n kube-system
k create clusterrolebinding argocd --clusterrole=cluster-admin --serviceaccount=kube-system:argocd
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | (grep argocd || echo "$_") | awk '{print $1}') | grep token: | awk '{print $2}'
```

Then, it is necessary to get the CA root certificate.
```
# test cluster

kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}" | base64
```

Note, that this needs to be base64 encoded based on the documentations provided from ArgoCD. Modify the `argocd.yml` file with the required token and CA. Additionally, the server should point to the control plane on the test cluster.

Now deploy ArgoCD with the modified values:
```
# production cluster

helm repo add argo https://argoproj.github.io/argo-helm
helm install my-argo-cd argo/argo-cd -f argocd.yml
```

## Install Ingress NGINX
It is necessary to also install a load balancer that is responsible for exposing the services of the cluster outside. This can be accomplished with Ingress NGINX.

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-ingress-nginx ingress-nginx/ingress-nginx -f ingress-nginx.yml
```

Where the services are exposed:
```
  "8000": "default/kubernetes-dashboard-web:8000"
  "8080": "default/gitea-http:3000"
  "8082": "default/gitea-ssh:22"
  "9000": "default/my-argo-cd-argocd-server:80"
  "9080": "default/kube-prometheus-grafana:80"
```

And can be modified accordingly depending on operator needs. Multiple replicas are established across separate nodes.

## Known Issues
| Symptoms | Remedy |
| -------- | ------ |
| Gitea services enter random crashloopbackoff | These seem to be a result of the subcharts that are being used with the Gitea Helm chart. The only suitable remedy is to redeploy the pod. |
| ArgoCD can't upgrade due CRD errors | **<WARNING!!>** The only suitable fix is to either ignore the CRD errors or delete it manually. Please exercise caution when doing this, as it can ruin the entire cluster!! |
| Pods stuck in pending, no PVC | You did not properly setup a dynamic provisioning suitable environment, or the storage class is not set as default. This document follows the rancher.io [local-path-provisioner](#install-local-path-on-both-clusters). |
