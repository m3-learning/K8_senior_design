# Gitlab install guide
The purpose of this project is to spinup a gitlab service for kubernetes utilizing local storage provisioners

## Disclaimer
The installation procedure gives you a sample-working production environment for GitLab. For best practices, please ensure that proper methods are used to secure each service. For any values specified in the configurations, they can be modified to the operator's specs.

## Dependencies
* GitLab Helm chart dependency
* Postgresql Helm chart dependency
* Redis Helm chart dependency
* Paths specified in volume should be satisfied across hosts

## Order of install operation
Production environments specified on GitLab

## Install Prerequiste
You must satisfy the `storage-class` requirements that may not be created already. To do this, supply the following command:
```
kubectl apply -f storage.yml
```

Then you must allocate volumes required for postgresql:
```
kubectl apply -f postgresql-volume.yml
```

Force patch the claim to the volume:
```
kubectl patch pvc data-my-postgresql-0 -p '{"spec":{"volumeName":"example-pv"}}'
```

This creates the storageclass necessary for the GitLab cluster and dependency services to use. Then, proceed to create the postgresql PVC. This is required since automated bounding does not happen for non-dynamic provisioners:
```
kubectl apply -f patch-postgresql-pvc-claim.yml
```

Now, postgresql can be deployed with the values.yml file supplied. First, the configurations are set such that it does not allow any dynamic provisioners to be set by default. This is shown for entries that consist of `"-"`. Then, postgresql is forced to use the `local-storage` storage class that was previously deployed in `storage.yml`. The most important variable set is that `persistance.existingClaim` is set to the volume claim set earlier. This can be applied by:
```
helm install postgresql bitnami/postgresql -f postgresql-values.yml
```

The next dependency required is redis, which similar to the previous deployment of postgresql, can be done. Note, the claim must be created for redis, and the `persistance.existingClaim` must also be set alongside disabling any dynamic provisioners. This is already supplied within the `patch-redis-pvc-claim.yml`. The following can be used to spin up a redis service:
```
kubectl apply -f redis-storage.yml
kubectl apply -f redis-volume.yml
kubectl apply -f redis-pvc-claim.yml
kubectl patch pvc data-my-redis-0 -p '{"spec":{"volumeName":"example-pv-redis"}}'
helm install redis bitnami/redis -f redis-values.yml
```

Now, we have setup the required dependencies for a bare minimum for a production-ready environment for GitLab.

## Prerequisite installs for GitLab
Patch the core storage class for GitLab
```
kubectl patch storageclass CUSTOM_STORAGE_CLASS_NAME -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Create persistence storageclass, volumes, and claims for Gitaly, MinIO, and Redis
