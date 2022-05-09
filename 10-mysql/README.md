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

## Presslabs Operator

* [Upstream Repository](https://github.com/presslabs/mysql-operator)
* [ArtifactHub.io - mysql-operator](https://artifacthub.io/packages/helm/presslabs/mysql-operator)

```shell
kubectl create namespace mysql
helm repo add presslabs https://presslabs.github.io/charts
helm upgrade --install --namespace mysql mysql-operator presslabs/mysql-operator --version=0.5.0-rc.3
```

## MySQL Cluster

```shell
kubectl apply -f presslabs-secret.yaml
kubectl apply -f presslabs-cluster.yaml
kubectl -n mysql get pvc
kubectl -n mysql run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h my-cluster-mysql -uroot -pnot-so-secure
```

## also check Tolerations

```shell
kubectl -n mysql describe pod ...
```
