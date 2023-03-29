# MySQL

* Create additional node deployment consisting of 3 nodes
  * Select a local storage flavor
  * Node labels
    * Key: mysql
    * Value: true
  * Node taints
    * Key: mysql
    * Value: true
    * Effect: NoSchedule

## Local Storage

* [Upstream Repository](https://github.com/rancher/local-path-provisioner)
* [local-path-provisioner Chart](https://github.com/rancher/local-path-provisioner/tree/master/deploy/chart)

```shell
kubectl create namespace local-path-storage
git clone https://github.com/rancher/local-path-provisioner.git
helm upgrade --install --namespace local-path-storage local-path-storage ./local-path-provisioner/deploy/chart/local-path-provisioner/ -f values-local-path-provisioner.yaml
kubectl get sc
```

## MySQL DB Operator

* [Upstream Repository](https://github.com/bitpoke/mysql-operator)
* [ArtifactHub.io - mysql-operator](https://artifacthub.io/packages/helm/bitpoke/mysql-operator)

```shell
kubectl create namespace mysql
helm repo add bitpoke https://helm-charts.bitpoke.io
helm repo update
helm upgrade --install --namespace mysql mysql-operator bitpoke/mysql-operator --version=0.5.3
```

## MySQL Cluster

```shell
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql-cluster.yaml
kubectl -n mysql get pvc
kubectl -n mysql run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h my-cluster-mysql -uroot -pnot-so-secure
```

## also check Tolerations

```shell
kubectl -n mysql describe pod
```
