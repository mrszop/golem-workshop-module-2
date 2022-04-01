# Init Container

* Let's create a namespace for our needs and switch into it.

```shell
kubectl create namespace <YOURNAME>
kubectl label namespace <YOURNAME> golem-workshop=true
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* Lets create a Pod with three containers. The first container is our main container and the second and third are init containers that wait until some service will show up

```shell
kubectl apply -f myapp.yaml
kubectl get -f myapp.yaml
```

* Lets see what our Pod is doing

```shell
kubectl describe -f myapp.yaml
kubectl logs myapp-pod -c init-myservice # Inspect the first init container
kubectl logs myapp-pod -c init-mydb      # Inspect the second init containers
```

* Now we will create the missing services so that our init containers can finaly start

```shell
kubectl apply -f services.yaml
```

* Check that the pod is now running

```shell
kubectl get -f myapp.yaml
```

* Lets tear down everything

```shell
kubectl delete -f myapp.yaml -f services.yaml
```
