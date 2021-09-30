# Production-grade Deployments

* Deploy application
  * We deploy a simple Apache with PHP that delivers an index.php on request and thus creates some load. Let's create a namespace for our needs and switch into it. With the next command we will create a skeleton that we will customize.

```shell
kubectl create namespace "YOUR_NAMESPACE"
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* Create a skeleton deployment file, apply it, check it, delete the deployment

```shell
kubectl -n create deployment --image=gcr.io/google_containers/hpa-example php-apache -o yaml --dry-run=client > php-apache-deployment-skeleton.yaml

kubectl -n apply -f php-apache-deployment-skeleton.yaml

kubectl -n get -f php-apache-deployment-skeleton.yaml

kubectl -n delete -f php-apache-deployment-skeleton.yaml
```

* Customize to our liking by defining resources and releasing a port

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: php-apache
  name: php-apache
  namespace: djarosch
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
            cpu: 50m
          limits:
            memory: 64Mi
            cpu: 50m
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
          exec:
            command:
            - cat
            - /var/www/html/index.php
          initialDelaySeconds: 5
          periodSeconds: 5
```

* Apply deployment and check if everything works as expected

```shell
kubectl -n apply -f php-apache-deployment.yaml
kubectl -n get deployment
kubectl -n describe deployment php-apache
```

* We need a service, with a clusterIP, so that the pods are response to our calls 
  * we can use the skeleton method to create a service 

```shell
kubectl -n create service clusterip php-apache --tcp=80:80 --dry-run=client -o yaml > php-apache-service-skeleton.yaml
kubectl -n apply -f php-apache-service.yaml
```

* Check if service is up as expected

```shell
kubectl -n get svc php-apache -o wide
kubectl -n describe svc php-apache
```

* Look out for Endpoints, thats you Pods!

```shell
kubectl -n get pods -o wide
```

* Create Horizontal Pod Autoscaler
  * we can us the skeleton method again and modify the YAML to our needs

```shell
kubectl -n autoscale deployment php-apache --cpu-percent=20 --min=1 --max=10 -o yaml --dry-run=client > php-apache-hpa-skeleton.yaml
```

* lets scale down our deployment

```shell
kubectl -n scale -f php-apache-deployment-skeleton.yaml --replicas=1
```

* Let's check the content of the HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: php-apache
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 20
```

* Apply the HorizontalPodAutoscaler

```shell
kubectl -n apply -f php-apache-hpa-skeleton.yaml
```

* Start a new pod with the busybox image which we will use to stress our service. We have to be a little bit patient until the results shows up

```shell
kubectl -n run -i --tty load-generator --image=busybox /bin/sh # if you run a brand new Pod or
kubectl -n attach load-generator -c load-generator -i -t # if you want to use it again after you exit it
while true; do wget -q -O- http://php-apache.YOUR-NAMESPACE.svc.cluster.local; done
```

* Alternative to busybox image via port-forward
  * Lets the local machine forward to the cluster. Either to a pod directly or to a service. To test pods, Bret Fisher's shpod.yml is also useful. You need to run the scenario with two windows

```shell
kubectl -n port-forward svc/php-apache 8080:80
kubectl -n  logs -f -l name=php-apache --all-containers
```

* Lets check what's going on

```shell
kubectl -n get hpa
kubectl -n top pod
kubectl -n get deployment php-apache
```

* Apply PodDisruptionBudget resource

```shell
kubectl -n apply -f php-apache-pdb.yaml
```

* Delete HPA, scale down to own replica and find out what node last Pod is runnning
  
```shell
kubectl -n apply -f php-apache-pdb.yaml
kubectl -n delete -f php-apache-hpa-skeleton.yaml
kubectl -n scale -f php-apache-deployment-skeleton.yaml --replicas=1
kubectl -n delete pod load-generator
kubectl -n get pod -o wide
kubectl -n get node
```

* try to drain node
  
```
kubectl -n drain "NODE-NAME"
```

* Tear down everything

```shell
kubectl delete -f php-apache-hpa.yaml php-apache-service.yaml php-apache-deployment.yaml php-apache-pdb.yaml
```
