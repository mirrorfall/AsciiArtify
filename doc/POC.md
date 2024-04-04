## Proof of Concept

`Ціль:` Команда погодилась з вашими аргументами та запросили підготувати Proof of Concept (PoC) по розгортанню GitOps-системи на рекомендованому вами варіанті Kubernetes. Командою запроповано продукт ArgoCD.

PoC — це етап, коли розробники перевіряють, чи є технічна можливість реалізувати концепцію продукту. На цьому етапі розробники використовують мінімальний набір функцій, щоб продемонструвати, що продукт може працювати та виконувати свої основні функції.

Вам потрібно практично розгорнути Kubernetes кластер за допомогою інструменту, що затверджений на етапі Concept. Встановити систему та налаштувати доступ команди до графічного інтерфейсу ArgoCD.

![ArgoCD](.img/argocd_arch.png)  

`ArgoCD` implements a GitOps approach using the Git repository as the source of truth to determine the desired state of the application. Kubernetes manifests can be specified in several ways: 
- [kustomize](https://kustomize.io/) applications  
- [helm](https://helm.sh/) charts
- [jsonnet](https://jsonnet.org/) files
- Plain directory of YAML/json manifests  

`ArgoCD` - is a Kubernetes controller that continuously monitors running applications and compares the current state with the desired one. A deployment whose current state differs from the target is considered 'out of sync'. ArgoCD informs and visualizes the differences, providing opportunities for automatic or manual synchronization of the desired state.

1. Prepare a separate local cluster for installing ArgoCD:
```bash
$ k3d cluster create argo
... 
INFO[0029] Cluster 'argo' created successfully!         
INFO[0029] You can now use it like this: kubectl cluster-info

$ kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:60765
CoreDNS is running at https://0.0.0.0:60765/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:60765/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

$ kubectl version
$ kubectl get all -A
```
2. Installation.
ArgoCD can be installed using `Helm', but now we will use a script from the official repository [ArgoCD](https://argo-cd.readthedocs.io/en/stable/#quick-start).
Create a namespace in which the system will be installed, then use the script for installation and check the state of the system after:
```bash
$ kubectl create namespace argocd
namespace/argocd created
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created

$ kubectl get all -n argocd

NAME                                                   READY   STATUS    RESTARTS   AGE
pod/argocd-redis-69f8795dbd-c2fwr                      1/1     Running   0          32s
pod/argocd-applicationset-controller-87bc8f4dd-4ctpt   1/1     Running   0          32s
pod/argocd-notifications-controller-74b56c5748-4tk9h   1/1     Running   0          32s
pod/argocd-dex-server-bd798767-l9djb                   1/1     Running   0          32s
pod/argocd-repo-server-79c9747d79-kmq5w                1/1     Running   0          32s
pod/argocd-application-controller-0                    1/1     Running   0          32s
pod/argocd-server-c9b448778-h476r                      1/1     Running   0          32s

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.43.85.9      <none>        7000/TCP,8080/TCP            32s
service/argocd-dex-server                         ClusterIP   10.43.46.16     <none>        5556/TCP,5557/TCP,5558/TCP   32s
service/argocd-metrics                            ClusterIP   10.43.119.63    <none>        8082/TCP                     32s
service/argocd-notifications-controller-metrics   ClusterIP   10.43.3.248     <none>        9001/TCP                     32s
service/argocd-redis                              ClusterIP   10.43.113.198   <none>        6379/TCP                     32s
service/argocd-repo-server                        ClusterIP   10.43.70.13     <none>        8081/TCP,8084/TCP            32s
service/argocd-server                             ClusterIP   10.43.67.28     <none>        80/TCP,443/TCP               32s
service/argocd-server-metrics                     ClusterIP   10.43.196.124   <none>        8083/TCP                     32s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-redis                       1/1     1            1           32s
deployment.apps/argocd-applicationset-controller   1/1     1            1           32s
deployment.apps/argocd-notifications-controller    1/1     1            1           32s
deployment.apps/argocd-dex-server                  1/1     1            1           32s
deployment.apps/argocd-repo-server                 1/1     1            1           32s
deployment.apps/argocd-server                      1/1     1            1           32s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-redis-69f8795dbd                      1         1         1       32s
replicaset.apps/argocd-applicationset-controller-87bc8f4dd   1         1         1       32s
replicaset.apps/argocd-notifications-controller-74b56c5748   1         1         1       32s
replicaset.apps/argocd-dex-server-bd798767                   1         1         1       32s
replicaset.apps/argocd-repo-server-79c9747d79                1         1         1       32s
replicaset.apps/argocd-server-c9b448778                      1         1         1       32s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     32s
```
3. Get access to ArgoCD GUI 
[The access can be retrieved](https://argo-cd.readthedocs.io/en/stable/getting_started/#3-access-the-argo-cd-api-server) in a few ways:  
- Service Type Load Balancer 
- Ingress
- Port Forwarding
Use `Port Forwarding` with the local port 8080.
```bash
$ kubectl port-forward svc/argocd-server -n argocd 8080:443&
[1] 38317
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
```
ArgoCD works with https by default, so when we try to open [127.0.0.1:8080](https://127.0.0.1:8080/) we get an error related to certificate. Therefore, in production, you need to fix that error.


4. Get Password and Log in
Use the command to get the password, specify the secret file `argocd-initial-admin-secret` and the output format `jsonpath="{.data.password}"`. 
This will return us a base64 encoded password, after which we use the `base64 -d` command to return the password to plain text. Enter the received password and `admin` login into the ArgoCD Web interface
```bash                                                                                                   
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"|base64 -d;echo
NKsWip85pAs17Ye0
```