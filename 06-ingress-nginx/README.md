# nginx Ingress Controller

**This installation is required only once per cluster.**

[ArtifactHub - ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

* Install nginx ingress controller

## Create Namespace called ingress-nginx

```shell
kubectl create namespace ingress-nginx
```

## Add Ingress NGINX Helm Repo and update Cache

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

## Install Ingress NGINX Controller with provided values.yaml, newest version

```shell
helm upgrade --install -f values.yaml --namespace ingress-nginx ingress-nginx ingress-nginx/ingress-nginx --version=4.3.0
```

## Check if Ingress NGINX Controller is up and running

```shell
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
```
