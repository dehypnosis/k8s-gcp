# GKE 기반 리소스 설치 이력

## 1. 우선 사항
- 로컬에 Helm 클라이언트 설치 및 GKE에 Tiller (Helm 서버 에이전트) 배포
```
# for non-macOS; https://github.com/helm/helm#install
$ brew install kubernetes-helm
...

$ helm init && helm repo update
...

# if Tiller in k8s cluster has not initialized, make Tiller authorized with previleged service account:
$ kubectl create serviceaccount -n kube-system tiller
$ kubectl create clusterrolebinding tiller-binding --clusterrole=cluster-admin --serviceaccount kube-system:tiller
$ helm init --service-account tiller
```

## 2. 설치 내역
- [NATS](./nats/README.md)
  - 클러스터에 **Helm** 에이전트 설치 및 권한 부여
  - `dev/prod` 네임스페이스 및 네임스페이간 네트워크 정책 생성
  - `nats` 네임스페이스에 **NATS** 서비스 (`dev/prod.nats.svc.cluster.local`) 배포
- [REDIS](./redis/README.md)
  - `redis` 네임스페이스에 **Redis** 서비스 (`dev/prod.redis.svc.cluster.local`) 배포
    - 추후 GCP의 Cloud MemoryStore로 전환을 권장:
    전환시 Cloud MemoryStore 인스턴스를 `strix-vpc`에 피어링하여 생성하고, 기존 `dev/prod.redis.svc.cluster.local` 를 Cloud MemoryStore의 IP를 가리키는 ExternalName 서비스로 변경한 뒤, 기존에 배포된 Redis를 제거.
- [NGINX](./nginx/README.md)
  - `nginx` 네임스페이스에 `nginx-ingress` 로드밸런서 서비스 `nginx-ingress-controller.nginx.svc.cluster.local` 배포
  - `cert-manager` 배포
