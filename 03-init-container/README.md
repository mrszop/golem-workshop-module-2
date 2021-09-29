# Init Container

* Let's create a namespace for our needs and switch into it.

```shell
kubectl create namespace "YOUR_NAMESPACE"
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* Lets create a Pod with three containers. The first container is our main container and the second and third are init containers that wait until some service will show up

```shell
kubectl -n apply -f myapp.yaml
kubectl -n get -f myapp.yaml
```

* Lets see what our Pod is doing

```shell
kubectl -n describe -f myapp.yaml
kubectl -n logs myapp-pod -c init-myservice # Inspect the first init container
kubectl -n logs myapp-pod -c init-mydb      # Inspect the second init containers
```

* Now we will create the missing services so that our init containers can finaly start

```shell
kubectl -n apply -f services.yaml
```

* Lets tear down everything

```shell
kubectl -n delete -f myapp.yaml services.yaml
```
