# Deployment with Ingress and TLS

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  kubectl create namespace <YOURNAME>
  kubectl config set-context --current --namespace="${YOURNAME}"
  ```

* Clone this repository to your working station and change into the directory for this exercise

---

## Exercise

Deploy a web application that makes use of external-dns, cert-manager and ingress-nginx.

* Edit hostnames in `web-application/deployment/ingress.yaml`

* Deploy application

  ```shell
  kubectl apply -f web-application/deployment/web-application.yaml
  ```

* Deploy Ingress

  ```shell
  kubectl apply -f web-application/deployment/ingress.yaml
  ```

## Verify

* Watch the logs of external-dns, cert-manager and ingress-nginx

  ```shell
  kubectl -n external-dns logs -f deployments/external-dns
  
  kubectl -n cert-manager logs -f deployments/cert-manager
  
  kubectl -n ingress-nginx logs -f deployments/ingress-nginx-controller
  ```

* for ingress-nginx please notice the log line:

  ```shell
  Error getting SSL certificate ... local SSL certificate ... was not found. Using default certificate
  ```

## Conclusion

* the web application in your personal namespace is now fully integrated into cert-manager, external-dns and ingress-nginx
* you can control certificate, dns entries and http routes completely through k8s resources
