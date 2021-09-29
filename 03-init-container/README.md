# Init Container

* Lets create a Pod with three containers. The first container is our main container and the second and third are init containers that wait until some service will show up

```shell
kubectl -n "${YOUR_NAMESPACE}" apply -f myapp.yaml
kubectl -n "${YOUR_NAMESPACE}" get -f myapp.yaml
```

* Lets see what our Pod is doing

```shell
kubectl -n "${YOUR_NAMESPACE}" describe -f myapp.yaml
kubectl -n "${YOUR_NAMESPACE}" logs myapp-pod -c init-myservice # Inspect the first init container
kubectl -n "${YOUR_NAMESPACE}" logs myapp-pod -c init-mydb      # Inspect the second init containers
```

* Now we will create the missing services so that our init containers can finaly start

```shell
kubectl -n "${YOUR_NAMESPACE}" apply -f services.yaml
```

* Lets tear down everything

```shell
kubectl -n "${YOUR_NAMESPACE}" delete -f myapp.yaml services.yaml
```
