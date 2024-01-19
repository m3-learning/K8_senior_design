# Kubernetes
Simple Ansible role to deploy kubernetes cluster

## Dependencies
* Ansible > 2.9
* Python3
* Ubuntu 20.04+

## Sample playbook
```
- hosts: all
  tasks:
    - name: Run kubernetes
      include_role:
        name: kubernetes
```

## Considerations
This role will be left here as reference, but generally we will be working with Kubespray as a better alternative.
