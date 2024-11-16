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


## Proxy para API Kubernetes
kubectl proxy --port=8080

http://localhost:8080/api/v1

http://localhost:8080/api/v1/namespaces/default/services/goserver-service

## Info
Todas configs ficam na pasta Kube/config

kubectl config get-clusters

kubectl config use-context `cluster name`



## Kind

kind create cluster

kind create cluster --config=k8s/kind.yaml --name=fullcycle

kind delete

kind get

https://kind.sigs.k8s.io/

kind get clusters

kind delete clusters kind


## Docker

docker build -t nfandre/hello-go . 

docker run --rm -p 80:80 nfandre/hello-go 

docker push nfandre/hello-go      

