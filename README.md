# kubernetes
https://kubernetes.io/docs/tasks/tools/

> brew install kubectl

kubectl cluster-info --context kind-kind

kubectl get nodes

kubectl config

kubectl apply -f k8s/pod.yaml

kubectl port-forward pod/goserver 8000:80

kubectl delete pod goserver

kubectl describe pod `pode name`

## Hierarquia kubernetes
Deployment > ReplicaSet > Pod


## ReplicaSet

kubectl apply -f k8s/replicaset.yaml

kubectl get replicasets

kubectl delete replicaset goserver

- Para o replicaSet atualizar a versão da imagem de um pod, é necessário deletá o pod para ser recriado com a imagem atualizada.
    - Para isso é utilizado o Deployment

## Deployment

kubectl apply -f k8s/deployment.yaml

kubectl get deployment
> goserver-56d7b8986d-57p9z 
> name deployment - Random name replicaSet - random name pod

kubectl describe deployment goserver

kubectl port-forward deployment/goserver 8000:80

## Rollout and Revisions

kubectl rollout history deployment `deployment name`

kubectl rollout undo deployment goserver 

kubectl rollout undo deployment `deployment name` --to-revision=1


## Service ("Load balancer")

Types:
- ClusterIp - gera ip interno
- NodePort - gera ip acessível para acesso de fora do k8s
  > forma "Arcaica", é possível configurar um range de porta para um service
```yaml
    - name: goserver-service
    port: 80
    protocol: TCP
    nodePort: 30001
``` 
  
- LoadBalancer
 > Possui ClusterIp, nodePort, Ip exclusivo
- Headless service

kubectl apply -f k8s/service.yaml

kubectl get services 
> kubectl get svc

kubectl port-forward svc/goserver-service 8000:80

### Target port vs Port
targetPort: Porta do container
port: Porta da service

##   Config objects (Environment, configs, passwords, sensitive data etc)
Using env: 
```yaml
    spec:
      containers:
        - name: goserver
          image: nfandre/hello-go:v4
          env: 
            - name: NAME
              value: "Andre"
            - name: AGE
              value: "36"
```

### ConfigMap
Using env with config map
```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: goserver-env
  data:
    NAME: "Andre"
    AGE: "24"
```

```yaml
  env: 
    - name: NAME
      valueFrom:
        configMapKeyRef:
          name: goserver-env
          key: NAME
```

Using envFrom (set all keys on environments variables):
```yaml
  envFrom:
    - configMapRef:
        name: goserver-env
```

kubectl apply -f k8s/configmap-env.yaml

kubectl apply -f k8s/configmap-family.yaml

### Inject ConfigMap on application (transform configmap using volume)
```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: goserver-env
  data:
    NAME: "Andre"
    AGE: "24"
```
Config deployment

```yaml
    spec:
      containers:
        - name: goserver
          image: nfandre/hello-go:v5.2
          envFrom:
            - configMapRef:
                name: goserver-env
          volumeMounts:
            - mountPath: "/go/myfamily"
              name: config
      volumes:
        - name: config
          configMap:
            name: configmap-family
            items:
              - key: members
                path: "family.txt"
```

kubectl exec -it goserver-d7ffb8769-w8phb   -- bash

kubectl logs goserver-d7ffb8769-w8phb 

kubectl get events --sort-by=.metadata.creationTimestamp


### Secrets
File config secret
```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: goserver-secret
  type: Opaque
  data:
    USER: "d2VzbGV5Cg=="
    PASSWORD: "MTIzNDU2Cg=="
```

kubectl apply -f k8s/secret.yaml

## Probes (Health check, Liveness e readiness)

### Liveness Probe
It checks if the container is health periodically 

Types: HTTP, COMMAND and TCP
```yaml
  spec:
  containers:
    - name: goserver
      image: nfandre/hello-go:v5.5
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8000 #container port
        periodSeconds: 5
        failureThreshold: 3
        timeoutSeconds: 1
        successThreshold: 1
```

kubectl apply -f k8s/deployment.yaml && watch -n1 kubectl get pods

### Readiness Probe
Verify when  application is ready to receive traffic

```yaml
  readinessProbe:
    httpGet:
      path: /healthz
      port: 8000 #container port
    periodSeconds: 3
    initialDelaySeconds: 10
```

### Startup Probe

```yaml
  startupProbe:
    httpGet:
      path: /healthz
      port: 8000
    periodSeconds: 3
    failureThreshold: 30
```

## Resource e Horizontal Pod Autoscaling (HPA)

### Install Metrics-server
repo: https://github.com/kubernetes-sigs/metrics-server

wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

add insecure tls to run on kind:
> --kubelet-insecure-tls


### Kubernetes Api server
kubectl get apiservices

### Deployments com Resources (cpy/ memory)

kubectl top pod goserver-68d4cc46b-k7v7s

### HPA with CPU
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: goserver-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    name: goserver
    kind: Deployment
  minReplicas: 1
  maxReplicas: 30
  targetCPUUtilizationPercentage: 25
```

kubectl apply -f k8s/hpa.yaml

kubetl get hpa

## Proxy para API Kubernetes
kubectl proxy --port=8080

kubectl port-forward deployment/goserver 8000:80

http://localhost:8080/api/v1

http://localhost:8080/api/v1/namespaces/default/services/goserver-service

## Info
Todas configs ficam na pasta Kube/config

kubectl config get-clusters

kubectl config use-context `cluster name`



## Kind

kind create cluster

kind create cluster --config=k8s/kind.yaml --name=fullcycle

kubectl cluster-info --context kind-fullcycle

kind delete

kind get

https://kind.sigs.k8s.io/

kind get clusters

kind delete clusters kind


## Docker

docker build -t nfandre/hello-go . 

docker run --rm -p 80:80 nfandre/hello-go 

docker push nfandre/hello-go      


## Utils

echo "andre" | base64

brew install watch

brew install wget

