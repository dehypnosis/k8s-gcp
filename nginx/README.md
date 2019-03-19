# Nginx Ingress Controller and Cert Manager with helm chart

This chart has been installed on GCP Kubernetes engine for Nginx Ingress to support ingress and automated issuerance of SSL certifications via let's-encrypt.

## 1. Services
- http://default-nginx-ingress-controller.nginx.svc.cluster.local

## 2. History
- Install Cert Manager: https://github.com/jetstack/cert-manager/tree/master/deploy/charts/cert-manager
```
# Create name space
$ kubectl apply -f 00-namespace-and-networkpolicy.yaml

## IMPORTANT: you MUST install the cert-manager CRDs **before** installing the
## cert-manager Helm chart
$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/00-crds.yaml

## IMPORTANT: if the cert-manager namespace **already exists**, you MUST ensure
## it has an additional label on it in order for the deployment to succeed
$ kubectl label namespace cert-manager certmanager.k8s.io/disable-validation="true"

## Create letsencrypt-prod/staging ClusterIssuer
$ kubectl apply -f 01-lets-encrypt-cluster-issuers.yaml
...

## Add the Jetstack Helm repository
$ helm repo add jetstack https://charts.jetstack.io

## Install the cert-manager helm chart
$ helm install --name cert-manager --namespace cert-manager jetstack/cert-manager \
   --set ingressShim.defaultIssuerName=letsencrypt-prod \
   --set ingressShim.defaultIssuerKind=ClusterIssuer \
   --set ingressShim.defaultACMEChallengeType=http01

NAME:   cert-manager
LAST DEPLOYED: Tue Mar 19 11:24:55 2019
NAMESPACE: cert-manager
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                    AGE
cert-manager-edit                       5s
cert-manager-view                       5s
cert-manager-webhook:webhook-requester  5s

==> v1/Pod(related)
NAME                                      READY  STATUS             RESTARTS  AGE
cert-manager-7ffb9f788b-k247v             0/1    ContainerCreating  0         3s
cert-manager-cainjector-67b4696847-m84fd  0/1    ContainerCreating  0         3s
cert-manager-webhook-6f58884b96-86sdp     0/1    ContainerCreating  0         4s

==> v1/Service
NAME                  TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)  AGE
cert-manager-webhook  ClusterIP  10.0.12.0   <none>       443/TCP  4s

==> v1/ServiceAccount
NAME                     SECRETS  AGE
cert-manager             1        5s
cert-manager-cainjector  1        5s
cert-manager-webhook     1        5s

==> v1alpha1/Certificate
NAME                              AGE
cert-manager-webhook-ca           4s
cert-manager-webhook-webhook-tls  4s

==> v1alpha1/Issuer
NAME                           AGE
cert-manager-webhook-ca        4s
cert-manager-webhook-selfsign  4s

==> v1beta1/APIService
NAME                                  AGE
v1beta1.admission.certmanager.k8s.io  4s

==> v1beta1/ClusterRole
NAME                     AGE
cert-manager             5s
cert-manager-cainjector  5s

==> v1beta1/ClusterRoleBinding
NAME                                 AGE
cert-manager                         4s
cert-manager-cainjector              5s
cert-manager-webhook:auth-delegator  5s

==> v1beta1/Deployment
NAME                     READY  UP-TO-DATE  AVAILABLE  AGE
cert-manager             0/1    1           0          4s
cert-manager-cainjector  0/1    1           0          4s
cert-manager-webhook     0/1    1           0          4s

==> v1beta1/RoleBinding
NAME                                                AGE
cert-manager-webhook:webhook-authentication-reader  4s

==> v1beta1/ValidatingWebhookConfiguration
NAME                  AGE
cert-manager-webhook  4s


NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://docs.cert-manager.io/en/latest/reference/issuers.html

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://docs.cert-manager.io/en/latest/reference/ingress-shim.html
```

- Install Nginx Ingress Controller: https://github.com/helm/charts/tree/master/stable/nginx-ingress
```
# Create name space
$ kubectl create ns nginx

# Install nginx
$ helm install stable/nginx-ingress --name nginx-ingress --namespace nginx \
   --set defaultBackend.enabled=false \
   --set controller.defaultBackendService=kube-system/default-http-backend

NAME:   nginx-ingress
LAST DEPLOYED: Tue Mar 19 11:28:02 2019
NAMESPACE: nginx
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                      DATA  AGE
nginx-ingress-controller  1     1s

==> v1/Pod(related)
NAME                                       READY  STATUS             RESTARTS  AGE
nginx-ingress-controller-6547669874-b9z26  0/1    ContainerCreating  0         0s

==> v1/Service
NAME                      TYPE          CLUSTER-IP  EXTERNAL-IP  PORT(S)                     AGE
nginx-ingress-controller  LoadBalancer  10.0.13.23  <pending>    80:31192/TCP,443:31307/TCP  0s

==> v1/ServiceAccount
NAME           SECRETS  AGE
nginx-ingress  1        1s

==> v1beta1/ClusterRole
NAME           AGE
nginx-ingress  1s

==> v1beta1/ClusterRoleBinding
NAME           AGE
nginx-ingress  1s

==> v1beta1/Deployment
NAME                      READY  UP-TO-DATE  AVAILABLE  AGE
nginx-ingress-controller  0/1    1           0          0s

==> v1beta1/Role
NAME           AGE
nginx-ingress  1s

==> v1beta1/RoleBinding
NAME           AGE
nginx-ingress  1s


NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace nginx get services -o wide -w nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

- Create DNS Record for Nginx Ingress Controller Load Balancer Service
  - Create A Record: "*.strix.co.kr" -> "35.203.175.188" in GCP Console
```
$ kubectl get all -n nginx
NAME                                            READY     STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-6547669874-b9z26   1/1       Running   0          43s

NAME                               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
service/nginx-ingress-controller   LoadBalancer   10.0.13.23   35.203.175.188     80:31192/TCP,443:31307/TCP   44s

NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-controller   1         1         1            1           44s

NAME                                                  DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx-ingress-controller-6547669874   1         1         1         44s
```

- Create Test Ingress
```
$ kubectl apply -f 02-test-ingress.yaml
...

...

...

# after a while
$ kubectl get certs --all-namespaces
NAMESPACE      NAME
cert-manager   cert-manager-webhook-ca
cert-manager   cert-manager-webhook-webhook-tls
kube-system    test-ingress-tls

$ kubectl describe -n kube-system certs test-ingress-tls
Name:         test-ingress-tls
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>
API Version:  certmanager.k8s.io/v1alpha1
Kind:         Certificate
Metadata:
  Creation Timestamp:  2019-03-19T02:50:51Z
  Generation:          1
  Owner References:
    API Version:           extensions/v1beta1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  Ingress
    Name:                  test-ingress
    UID:                   9990a07e-49f0-11e9-9b57-42010a8a0078
  Resource Version:        3396919
  Self Link:               /apis/certmanager.k8s.io/v1alpha1/namespaces/kube-system/certificates/test-ingress-tls
  UID:                     cea01980-49f1-11e9-9b57-42010a8a0078
Spec:
  Acme:
    Config:
      Domains:
        test-ingress.strix.co.kr
      Http 01:
  Dns Names:
    test-ingress.strix.co.kr
  Issuer Ref:
    Kind:       ClusterIssuer
    Name:       letsencrypt-prod
  Secret Name:  test-ingress-tls
Status:
  Conditions:
    Last Transition Time:  2019-03-19T02:51:19Z
    Message:               Certificate is up to date and has not expired
    Reason:                Ready
    Status:                True
    Type:                  Ready
  Not After:               2019-06-17T01:51:18Z
Events:
  Type    Reason              Age   From          Message
  ----    ------              ----  ----          -------
  Normal  Generated           14m   cert-manager  Generated new private key
  Normal  GenerateSelfSigned  14m   cert-manager  Generated temporary self signed certificate
  Normal  OrderCreated        14m   cert-manager  Created Order resource "test-ingress-tls-1826395262"
  Normal  OrderComplete       13m   cert-manager  Order "test-ingress-tls-1826395262" completed successfully
  Normal  CertIssued          13m   cert-manager  Certificate issued successfully

# now valid https connection works
```

- Enable Horizontal Pod AutoScaling in GCP Console