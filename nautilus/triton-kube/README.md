# triton-kube
Deploys NVIDIA Triton Inference Server onto a Kubernetes environment

## Dependencies
* kubectl

## Building
The configurations were built utilizing Kompose, and can be applied to the project:
```
kompose convert
```

## Deploying
```
kubectl apply -f tritonserver-claim0-persistentvolumeclaim.yaml
kubectl apply -f tritonserver-service.yaml
kubectl apply -f tritonserver-deployment.yaml
```

## Deploying Models
For applying the NVIDIA Triton Inference Server to utilize trained models, you must import the models to the `/models` directory of the deployment. This can be accomplished through kubectl:
```
kubectl cp <model_repository> <pod_name>:/models
```
The model structure consists of:
```
<model_name>
    ├── 1
    │   └── model.savedmodel
    │       ├── keras_metadata.pb
    │       ├── saved_model.pb
    │       └── variables
    │           ├── variables.data-00000-of-00001
    │           └── variables.index
    └── config.pbtxt
```

Then, redeploy the environment:
```
kubectl destroy -f tritonserver-deployment.yaml
kubectl apply -f tritonserver-deployment.yaml
```

## Communicating with the NVIDIA Triton Inference Server
The deployment exposes the ports: `8000`, `8001`, and `8002`, and they can be accessed through API calls. For more information, please see the NVIDIA documentations on interacting with the Triton Inference Server.

# Sample Project: MNIST
The instructions for deploying MNIST utilizes another project, `triton-mnist-example` which is authored by niyazed, and can be found [here](https://github.com/niyazed/triton-mnist-example).

## Dependencies:
* Python 3.11
* kubectl
* Access to NVIDIA GPU for NVIDIA Triton Inference Server

## Copy The Model
The MNIST model can be transferred to the NVIDIA Triton Inference Server:
```
kubectl cp model_repository/mnist_model <pod_name>:/models
```

Then, ensure to expose the ports required:
```
kubectl port-forward pod/<pod_name> <local_port>:8000
kubectl port-forward pod/<pod_name> <local_port>:8001
kubectl port-forward pod/<pod_name> <local_port>:8002
```

Finally, the python script can be utilized:
```
python3 -m pip install -r requirements.txt
python3 triton-infer.py
```

## Authors
-------
* Dexter Le (dql27@drexel.edu)
