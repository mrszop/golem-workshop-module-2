# External DNS

**This installation is required only once per cluster.**

[ArtifactHub - external-dns](https://artifacthub.io/packages/helm/bitnami/external-dns)

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

* Replace all occurrences of `CHANGEME` in values.yaml with the provided secrets

* Install external-dns with Helm:

```shell
kubectl create namespace external-dns
```

```shell
helm upgrade --install external-dns --namespace external-dns --values values.yaml bitnami/external-dns --version=5.4.7
```

* Check if ExternalDNS Pod is up and running

```shell
kubectl --namespace=external-dns get pods -l "app.kubernetes.io/name=external-dns,app.kubernetes.io/instance=external-dns"
```

* Check ExternalDNS Pod Logs

```shell
kubectl --namespace=external-dns logs -l "app.kubernetes.io/name=external-dns,app.kubernetes.io/instance=external-dns"
```

* Deploy nginx test with dns entry:

```shell
kubectl apply -f nginx-test.yaml --namespace external-dns
```

* Check Deployment, Services and Pods from previous Deployment and watch out for External IP for Service `service/nginx`

```shell
kubectl get deployment,pods,services
```

* Should look like this

```shell
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
service/nginx          LoadBalancer   10.240.26.62    185.56.130.197   80:32428/TCP   2m19s
```
