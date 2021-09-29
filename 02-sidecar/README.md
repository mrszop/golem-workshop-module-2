# Sidecar Container

* Let's create a namespace for our needs and switch into it.

```shell
kubectl create namespace "YOUR_NAMESPACE"
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* There are two examples for sidecar container management in this repository

* Lets apply our first manifest and port-forward to it

```shell
kubectl -n apply -f sidecar-echo.yaml
kubectl -n describe -f sidecar-echo.yaml
kubectl -n port-forward pod/sidecar-container-demo 8080:80
```

* Lets tear down everything

```shell
kubectl -n delete -f sidecar-echo.yaml
```

* In this example a sidecar proxy is used to provide basic authentication for an application without authentication itself. The proxy has an exposed container port on port 0.0.0.0:80 and is available to other pods. The application container has no container pods exposed and only listens for requests on the loopback device on port 127.0.01:8080

```shell
kubectl -n apply -f sidecar-proxy.yaml
kubectl -n port-forward deployment/nginx-deployment 8080:80
```

* Now we are creating a port-forward to our new application to check how a basic authentication could look like

```shell
curl localhost:8080
curl localhost:8080 -u admin:admin
```

* Lets tear down everything

```shell
kubectl -n delete -f sidecar-proxy.yaml
```
