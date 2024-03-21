# Sidecar Container

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

```shell
kubectl create namespace <YOURNAME>
kubectl config set-context --current --namespace="${YOURNAME}"
```

* Clone this repository to your working station and change into the directory for this exercise

---

## Exercise

* There are two examples for sidecar container management in this repository

## Exercise 1

* Lets apply our first manifest and port-forward to it

```shell
kubectl apply -f sidecar-echo.yaml
kubectl describe -f sidecar-echo.yaml
kubectl port-forward pod/sidecar-container-demo 8080:80
```

* Go to http://localhost:8080

---

### Clean up

* Lets tear down everything

```shell
kubectl delete -f sidecar-echo.yaml
```

---

## Exercise 2

* In the next example a sidecar proxy is used to provide basic authentication for an application without authentication itself. The proxy has an exposed container port on port 0.0.0.0:80 and is available to other pods. The application container has no container pods exposed and only listens for requests on the loopback device on port 127.0.01:8080

```shell
kubectl apply -f sidecar-proxy.yaml
kubectl port-forward deployment/nginx-deployment 8080:80
```

* Now we are creating a port-forward to our new application to check how a basic authentication could look like

```shell
curl localhost:8080
curl localhost:8080 -u admin:admin
```

---

### Clean up

* Lets tear down everything

```shell
kubectl delete -f sidecar-proxy.yaml
```
