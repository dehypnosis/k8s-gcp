# Redis Cluster with helm chart

This chart has been installed on GCP Kubernetes engine for REDIS to support caching.
Later, for the stable productions, remove `redis-prod` release and create [Redis Memory Store](https://console.cloud.google.com/memorystore/redis/locations/none/instances/new?project=strix-co-kr) supported by GCP self. Then replace existing ClusterIP service to ExternalName service which refers the created one.

## 1. Services
- redis://dev.redis.svc.cluster.local:6379
- redis://prod.redis.svc.cluster.local:6379
- redis://prod-slave.redis.svc.cluster.local:6379

## 2. History
- Install
```
# create redis namespace and network policy
$ kubectl apply -f 00-namespace-and-networkpolicy.yaml

# install stable/redis helm chart for dev
$ helm install --name redis-dev stable/redis --namespace redis -f ./01-dev-redis-helm-values.yaml

...

Redis can be accessed via port 6379 on the following DNS name from within your cluster:

   redis-dev-master.redis.svc.cluster.local

# update redis-dev network policy
$ kubectl get networkpolicy redis-dev -n redis -o yaml > ./temp.yaml
...
# edited networkpolicy; see yaml file.
...
$ kubectl apply -f 02-dev-redis-networkpolicy.yaml

# install stable/nats helm chart for prod
$ helm install --name redis-prod stable/redis --namespace redis -f ./03-prod-redis-helm-values.yaml

...

Redis can be accessed via port 6379 on the following DNS names from within your cluster:

   redis-prod-master.redis.svc.cluster.local for read/write operations
   redis-prod-slave.redis.svc.cluster.local for read-only operations

# update prod-nats network policy
$ kubectl get networkpolicy redis-prod -n redis -o yaml > ./temp.yaml
...
# edited networkpolicy; see yaml file.
...
$ kubectl apply -f 04-prod-redis-networkpolicy.yaml

# for the short url, make aliases for services

$ kubectl create service externalname dev -n redis --external-name redis-dev-master.redis.svc.cluster.local && \
   kubectl create service externalname dev-slave -n redis --external-name redis-dev-slave.redis.svc.cluster.local && \
   kubectl create service externalname prod -n redis --external-name redis-prod-master.redis.svc.cluster.local && \
   kubectl create service externalname prod-slave -n redis --external-name redis-prod-slave.redis.svc.cluster.local

service "dev" created
service "dev-slave" created
service "prod" created
service "prod-slave" created
```
