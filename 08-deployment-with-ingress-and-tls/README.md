# Deployment with Ingress and TLS

* Edit hostnames in `web-application/deployment/ingress.yaml`

* Deploy application

```shell
kubectl apply -f web-application/deployment/web-application.yaml
```

* Deploy Ingress

```shell
kubectl apply -f web-application/deployment/ingress.yaml
```
