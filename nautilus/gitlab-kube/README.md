# gitlab-kube
Deploys gitlab-ee onto a Kubernetes environment

## Dependencies
* kubectl

## Building
The configurations were built utilizing Kompose, and can be applied to the project:
```
kompose convert
```

## Deploying
```
kubectl apply -f gitlab-claim0-persistentvolumeclaim.yaml
kubectl apply -f gitlab-claim1-persistentvolumeclaim.yaml
kubectl apply -f gitlab-claim2-persistentvolumeclaim.yaml
kubectl apply -f gitlab-service.yaml
kubectl apply -f gitlab-deployment.yaml
```

## Authors
-------
* Dexter Le (dql27@drexel.edu)
