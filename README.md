# acodetest

thing for people

# goals

Deploy Argo CD and prepare a GitOps repo that makes Argo CD do the following:
1. Manage its own configurations/lifecycle
2. Deploy Prometheus and any required configurations to make it monitor Argo CD (with
dashboards and alerts)
3. Replace the Helm binary included with Argo CD with a different version of Helmand specifically use Helm version 3.7
4. Ensure you set a sync wave with a value of -10 on the app-of-apps
Share the git repo with us, and be prepared to discuss what you did and how it works

## assumptions

* helm is installed
* argocd cli is installed
* latest minikube is installed
* security is an afterthought
* mvp is the goal, not the most elegant solution

## setup steps

1. Start minikube
   ```
   minikube start
   ```
2. Add helm repo (assumes helm is installed) from https://github.com/argoproj/argo-helm
   ```
   helm repo add argo https://argoproj.github.io/argo-helm
   helm install argocd argo/argo-cd -n argocd --create-namespace
   ```
3. Wait a bit, you should see the Argo pods
4. Per helm output, you can forward the port then visit localhost:8080 to get the ArgoCD login
    ```
    kubectl port-forward service/argocd-server -n argocd 8080:443
    ```
5. Grab initial password from CLI or secret
    ```
    argocd admin initial-password -n argocd
    ```
6. Log into the ArgoCD UI, you will see zero apps
7. Apply the app of apps yaml from root of this git repo
    ```
    kubectl apply -f bootstrap/app-of-apps.yaml
    ```
    You will see a new app appear in ArgoCD, if done correctly you will see the app of apps and the argocd app connected/blow it.
8. After some time, you should see the app of apps, argocd, and monitoring apps up and green.

## check monitoring
   
Stuff is running, now lets verify.

### ArgoCD managing its own lifecycle 

Metric services are not enabled by default, we know its being managed because the metric services appear. Which means the Helm chart with values was used.

*kubectl get svc -n argocd*

Example:
```
kubectl get svc -n argocd
NAME                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
argocd-application-controller-metrics      ClusterIP   10.101.165.186   <none>        8082/TCP            8h
argocd-applicationset-controller           ClusterIP   10.107.217.255   <none>        7000/TCP            8h
argocd-applicationset-controller-metrics   ClusterIP   10.105.92.204    <none>        8080/TCP            8h
argocd-dex-server                          ClusterIP   10.109.99.137    <none>        5556/TCP,5557/TCP   8h
argocd-notifications-controller-metrics    ClusterIP   10.98.232.53     <none>        9001/TCP            8h
argocd-redis                               ClusterIP   10.111.249.189   <none>        6379/TCP            8h
argocd-repo-server                         ClusterIP   10.107.194.121   <none>        8081/TCP            8h
argocd-repo-server-metrics                 ClusterIP   10.107.83.156    <none>        8084/TCP            8h
argocd-server                              ClusterIP   10.102.200.179   <none>        80/TCP,443/TCP      8h
argocd-server-metrics                      ClusterIP   10.99.228.33     <none>        8083/TCP            8h

```

### Prometheus works with ArgoCD metrics and has an alert, plus Grafana

Prometheus UI can be viewed with a port forward

*kubectl port-forward svc/prometheus-operated -n monitoring  9090*

Then visit http://localhost:9090 and take a look, queries for argocd metrics like argocd_app_info should work. And two custom alerts were added named  ArgoCDAppOutOfSync and ArgoCDAppDegraded

For grafana, forward a port as you please, such as 

kubectl port-forward svc/monitoring-grafana   -n monitoring   8081:80

Then visit http://localhost:8081/ , the default user is admin, the password is in a secret. Retrieve it via:

```
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" -n monitoring | base64 --decode ; echo
```

An ArgoCD dashboard can be found with working metrics/data.

### Helm binary changed/updated in repo-server pod

## troubleshooting


## random notes

Using helm only I apparently found this bug that breaks repo server if you use wrong versions, in addition to setting the annotation limit (which is in starter guide anyway)

https://github.com/argoproj/argo-cd/issues/26595