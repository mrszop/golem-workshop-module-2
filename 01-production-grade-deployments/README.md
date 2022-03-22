# Production-grade Deployments

* Deploy application
  * We deploy a simple Apache with PHP that delivers an index.php on request and thus creates some loaad. Let's create a namespace for our needs and switch into it. With the next command we will create a skeleton that we will customize.

```shell
kubectl create namespace "YOUR_NAMESPACE"
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* Create a skeleton deployment file, apply it, check it, delete the deployment afterwards
* This is just for demo purposes to show how to generate a deployment
* Please check also https://k8syaml.com/

```shell
kubectl create deployment --image=gcr.io/google_containers/hpa-example php-apache -o yaml --dry-run=client > php-apache-deployment-skeleton.yaml # no need to delete creationTimestamp: null

kubectl apply -f php-apache-deployment-skeleton.yaml

kubectl get -f php-apache-deployment-skeleton.yaml

kubectl delete -f php-apache-deployment-skeleton.yaml

rm -f php-apache-deployment-skeleton.yaml
```

* Customize to our liking by defining resources and releasing a port (already done in `php-apache-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: php-apache
  name: php-apache
spec:
  replicas: 10
  selector:
    matchLabels:
      app: php-apache
  strategy:
    rollingUpdate:
      # wir möchten, bei einem Deployment, 100% aller Pods direkt neu erstellen
      maxSurge: 100%
      # wir möchten, bei einem Deployment, das maximal 50% aller Pods gleichzeitig down gehen
      maxUnavailable: 50%
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      affinity:
        # Pod soll NICHT auf und der selben Node gescheduled werden
        podAntiAffinity:
          # aber wenn er schon mal drauf läuft, dann egal
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              podAffinityTerm:
                # anhand folgender Pod Labels soll entschieden werden
                labelSelector:
                  matchExpressions:
                    - key: "app"
                      operator: In
                      values:
                        - php-apache
                # die Auswirkung soll auf die Node anhand ihres Namens erfolgen
                topologyKey: "kubernetes.io/hostname"
      containers:
      #- image: nginx
      - image: gcr.io/google_containers/hpa-example
        name: hpa-example
        resources:
          requests:
            memory: 64Mi
            cpu: 70m
          limits:
            memory: 64Mi
            cpu: 70m
        ports:
          - containerPort: 80
            name: http
        livenessProbe:
          exec:
            command:
            - cat
            - /var/www/html/index.php
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 1
          failureThreshold: 3
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

* Apply deployment and check if everything works as expected

```shell
kubectl apply -f php-apache-deployment.yaml
kubectl get -f php-apache-deployment.yaml
kubectl get pod,deploy
kubectl describe deployment php-apache
```

* We need a service, with a clusterIP, so that the pods are response to our calls 
  * we could use the skeleton method to create a service like before for deployment
    * `kubectl create service clusterip php-apache --tcp=80:80 --dry-run=client -o yaml > php-apache-service-skeleton.yaml`

```shell
kubectl apply -f php-apache-service.yaml
```

* Check if service is up as expected

```shell
kubectl get -f php-apache-service.yaml
kubectl get svc php-apache -o wide
```

* Look out for Endpoints. These are your Pods!
  
```sh
kubectl describe svc php-apache
```

```shell
kubectl get pods -o wide
```

* lets scale down our deployment

```shell
kubectl scale -f php-apache-deployment.yaml --replicas=1
kubectl get -f php-apache-deployment.yaml
kubectl get pods
```

* Let's check the content of the HorizontalPodAutoscaler in `php-apache-hpa.yaml`

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  # das Target bezieht sich auf den CPU Request des gesamten Pods
  # es ist von daher ein MUSS die Ressourcen Requests zu setzen, damit das funktioniert
  targetCPUUtilizationPercentage: 25
```

* Apply the HorizontalPodAutoscaler
  * we could use the skeleton method again and modify the YAML to our needs
    * `kubectl autoscale deployment php-apache --cpu-percent=20 --min=1 --max=10 -o yaml --dry-run=client > php-apache-hpa-skeleton.yaml`

```shell
kubectl apply -f php-apache-hpa.yaml
kubectl get -f php-apache-hpa.yaml
```

* Start a new pod with the busybox image which we will use to stress our service. We have to be a little bit patient until the results shows up

```shell
kubectl run -i --tty load-generator --image=busybox /bin/sh # if you run a brand new Pod or
kubectl attach load-generator -c load-generator -i -t # if you want to use it again after you exit it
while true; do wget -q -O- http://php-apache.YOUR-NAMESPACE.svc.cluster.local; done
```

* Alternative to busybox image via port-forward
  * Lets the local machine forward to the cluster. Either to a pod directly or to a service. To test pods, Bret Fisher's shpod.yml is also useful. You need to run the scenario with two windows

```shell
kubectl port-forward svc/php-apache 8080:80
kubectl logs -f -l name=php-apache --all-containers
```

* Lets check what's going on

```shell
kubectl get hpa
kubectl top pod
kubectl get deployment php-apache
```

* Apply PodDisruptionBudget resource

```shell
kubectl apply -f php-apache-pdb.yaml
```

* Delete HPA, scale down to own replica and find out what node last Pod is runnning
  
```shell
kubectl delete -f php-apache-hpa.yaml
kubectl scale -f php-apache-deployment.yaml --replicas=1
kubectl delete pod load-generator
kubectl get pod -o wide
kubectl get node
```

* try to drain node (this should only be done by one user)
  
```sh
kubectl drain "NODE-NAME" --ignore-errors --ignore-daemonsets --delete-emptydir-data # extra options because of metakube specific pods running
```

* Tear down everything

```shell
kubectl delete -f php-apache-hpa.yaml -f php-apache-service.yaml -f php-apache-deployment.yaml -f php-apache-pdb.yaml
```

* you should see a message like this
  * error when evicting pods/"php-apache-f966667ff-wshts" -n "YOUR_NAMESPACE" (will retry after 5s): Cannot evict pod as it would violate the pod's disruption budget.
