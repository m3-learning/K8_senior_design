# Test Environment Setup For Docker

The purpose of this directory is to create docker images with the required dependencies for setting up a High-Availability Kubernetes cluster and ElasticSearch locally.

## Table of Contents
------
1. [Requirements](#requirements)
2. [Build](#build)
3. [Run and Deploy](#run-and-deploy)
4. [Cleanup](#cleanup)
5. [Managing Multiple Environments](#managing-multiple-environments)
6. [Authors](#authors)

## Requirements
-------
The test environment requires that you have installed locally on your machine: **docker** and **docker compose**. For setup guides, **[please consult the manual here](https://docs.docker.com/engine/install/)**. A working SSH public key must also be supplied in this given directory, named as: `id_rsa.pub`.

## Build
-------
You must build the docker images if you make any changes to the `Dockerfile` or `docker-compose.yml`. This can easily be done:
```
docker compose build
```

## Run and Deploy
-------
You can spin up the nodes:
```
docker compose up -d
```

## Cleanup
-------
When finished, you can kill the nodes:
```
docker compose down
```

## Managing Multiple Environments
-------
When making any additions, please submit a separate branch with your changes. This should be ran against the GitHub actions pipelines to validate the nodes and ansible roles. Contributions to `test` should be generic and the minimum requirements to installing the necessary services.

## Authors
-------
* Dexter Le (dql27@drexel.edu)
