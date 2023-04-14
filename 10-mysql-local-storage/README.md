# MySQL

**This installation is required only once per cluster.**

## Prerequisites

* Create additional node deployment `local-storage-nodes` consisting of 3 nodes
  * Select a local storage flavor (`l1.small`)
  * Node labels
    * Key: mysql
    * Value: true
  * Node taints
    * Key: mysql
    * Value: true
    * Effect: NoSchedule

---

## Prepare Local Storage Nodes

* [Upstream Repository](https://github.com/rancher/local-path-provisioner)
* [local-path-provisioner Chart](https://github.com/rancher/local-path-provisioner/tree/master/deploy/chart)

  ```shell
  kubectl create namespace local-path-storage
  git clone https://github.com/rancher/local-path-provisioner.git
  helm upgrade --install --namespace local-path-storage local-path-storage ./local-path-provisioner/deploy/chart/local-path-provisioner/ -f values-local-path-provisioner.yaml
  kubectl get sc
  ```

---

## Install mysql DB Operator

* [Upstream Repository](https://github.com/bitpoke/mysql-operator)
* [ArtifactHub.io - mysql-operator](https://artifacthub.io/packages/helm/bitpoke/mysql-operator)

  ```shell
  kubectl create namespace mysql
  helm repo add bitpoke https://helm-charts.bitpoke.io
  helm repo update
  helm upgrade --install --namespace mysql mysql-operator bitpoke/mysql-operator --version=0.5.3
  ```

---

## Create MySQL cluster

* Create the mysql cluster and wait a while until the replicas have spun up completely

  ```shell
  kubectl -n mysql apply -f mysql-secret.yaml
  kubectl -n mysql apply -f mysql-cluster.yaml
  ```

* Meanwhile observe the cluster creation

  ```shell
  kubectl -n mysql get mysqlclusters.mysql.presslabs.org my-cluster
  
  # Expected result:
  NAME         READY   REPLICAS   AGE
  my-cluster   True    3          4m5s
  ```

* Verify the local storage nodes are used for persistent volume claims and volumes

  ```shell
  kubectl -n mysql get pvc
  ```

* Then connect to the mysql cluster for the first time

  ```shell
  kubectl -n mysql run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h my-cluster-mysql -uroot -pnot-so-secure
  
  # expected result: you are successfully connected to mysql
  # for testing, issue some SQL statement:
  mysql> show databases;
  ```

* check tolerations

  ```shell
  kubectl -n mysql describe pod
  ```

---

## Conclusion

We deployed a production-ready mysql database which persists data on local storage nodes.
This improves I/O-throughput for the database.

Be careful! Local storage with Kubernetes involves further tasks and caution when upgrading or restarting nodes. 
