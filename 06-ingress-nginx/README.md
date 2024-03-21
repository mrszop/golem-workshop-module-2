# Ingress Controller: ingress-nginx

**This installation is required only once per cluster.**

[ArtifactHub - ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

* Install nginx ingress controller

### Create namespace called ingress-nginx

```shell
kubectl create namespace ingress-nginx
```

### Add Ingress NGINX Helm Repo and update Cache

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### Install Ingress NGINX Controller with provided values.yaml, particular version

* Helm commands can be lengthy and confusing, so let's do a drill down:

Example syntax:
```shell
helm upgrade --install -f <VALUES-FILE> --namespace <NAMESPACE> <RELEASE> <REPOSITORY>/<CHART> --version=<VERSION>
```

Now let's install it:
```shell
helm upgrade --install -f values.yaml --namespace ingress-nginx ingress-nginx ingress-nginx/ingress-nginx --version=4.10.0
```

### Check if Ingress NGINX Controller is up and running

```shell
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
```

### Verify

* determine the EXTERNAL-IP of ingress-nginx loadbalancer service

  ```shell
  kubectl -n ingress-nginx get svc
  ```

* Visit the default backend
  * Browse http://EXTERNAL-IP or
  * `curl http://<EXTERNAL-IP>`

* Expected result:

  `default backend - 404`

---

## Conclusion

* You have now installed ingress-nginx as an ingress controller in your cluster
* it is a central entrypoint reachable from the internet through one public IP address
* it receives traffic and can forward requests to multiple endpoints
