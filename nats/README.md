# NATS Cluster with helm chart

This chart has been installed on GCP Kubernetes engine for NATS protocol support.
[NATS protocol](https://nats.io) is a simple text-based protocol which provides sync, async, topic based pub/sub (broadcasting, least-once or most-once feature).
Currently [moleculer protocol](https://moleculer.services/docs/0.13/api/protocol.html) based [services on GCP](https://github.com/strix-kr/mol-api) are using NATS protocol for communication.

## 1. Services
- nats://dev.nats.svc.cluster.local:4222
- http://dev-monitor.nats.svc.cluster.local:8222
- nats://prod.nats.svc.cluster.local:4222
- http://prod-monitor.nats.svc.cluster.local:8222

## 2. History

- Install
```
# create nats namespace and network policy between dev, prod, default, nats namespaces
$ kubectl apply -f 00-namespaces-and-networkpolicies.yaml

# install stable/nats helm chart for dev
$ helm install stable/nats --name nats-dev --namespace nats -f ./01-dev-nats-helm-values.yaml
...

NATS can be accessed via port 4222 on the following DNS name from within your cluster:

   nats-dev-nats-client.nats.svc.cluster.local

NATS monitoring service can be accessed via port 8222 on the following DNS name from within your cluster:

   nats-dev-nats-monitoring.nats.svc.cluster.local


# update nats-dev-nats network policy
$ kubectl get networkpolicy nats-dev-nats -n nats -o yaml > ./temp.yaml
...
# edited networkpolicy; see yaml file.
...
$ kubectl apply -f 02-dev-nats-networkpolicy.yaml

# install stable/nats helm chart for prod
$ helm install stable/nats --name nats-prod --namespace nats -f ./03-prod-nats-helm-values.yaml

...

NATS can be accessed via port 4222 on the following DNS name from within your cluster:

   nats-prod-nats-client.nats.svc.cluster.local

NATS monitoring service can be accessed via port 8222 on the following DNS name from within your cluster:

   nats-prod-nats-monitoring.nats.svc.cluster.local


# update prod-nats network policy
$ kubectl get networkpolicy nats-prod-nats -n nats -o yaml > ./temp.yaml
...
# edited networkpolicy; see yaml file.
...
$ kubectl apply -f 04-prod-nats-networkpolicy.yaml

# now can connect via nats://nats-dev-nats-client.nats.svc.cluster.local:4222, or nats://nats-prod-nats-client.nats.svc.cluster.local:4222
# also can see monitoring ui via nats://nats-prod-nats-client.nats.svc.cluster.local:4222

# but for the short url, make aliases for services
$ kubectl create service externalname dev -n nats --external-name nats-dev-nats-client.nats.svc.cluster.local \
 && kubectl create service externalname prod -n nats --external-name nats-prod-nats-client.nats.svc.cluster.local \
 && kubectl create service externalname dev-monitor -n nats --external-name nats-prod-nats-monitoring.nats.svc.cluster.local \
 && kubectl create service externalname prod-monitor -n nats --external-name nats-prod-nats-monitoring.nats.svc.cluster.local

service "dev" created
service "prod" created
service "dev-monitor" created
service "prod-monitor" created
```