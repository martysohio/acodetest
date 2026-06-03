# TOC

ArgoCD 24 hour bootcamp

- [TOC](#toc)
- [Goals](#goals)
- [Assumptions](#assumptions)
- [Setup steps](#setup-steps)
  - [Quick portforward copy paste](#quick-portforward-copy-paste)
- [Requirements](#requirements)
  - [Requirement 1 ArgoCD managing its own lifecycle](#requirement-1-argocd-managing-its-own-lifecycle)
  - [Requirement 2 Prometheus works with ArgoCD metrics and has an alert, plus Grafana](#requirement-2-prometheus-works-with-argocd-metrics-and-has-an-alert-plus-grafana)
  - [Requirement 3 Helm binary changed/updated in repo-server pod](#requirement-3-helm-binary-changedupdated-in-repo-server-pod)
- [Troubleshooting](#troubleshooting)
- [Random notes](#random-notes)



# Goals

Deploy Argo CD and prepare a GitOps repo that makes Argo CD do the following: 
1. Manage its own configurations/lifecycle 
2. Deploy Prometheus and any required configurations to make it monitor Argo CD (with 
dashboards and alerts) 
3. Replace the Helm binary included with Argo CD with a different version of Helm

# Assumptions

* helm is installed
* argocd cli is installed
* latest minikube is installed
* helm is installed (3.x )
* security is an afterthought
* mvp is the goal, not the most elegant solution, or even best practices

# Setup steps

1. Start minikube
   ```
   minikube start
   ```
2. Add helm repo (assumes helm is installed) from https://github.com/argoproj/argo-helm
   ```
   helm repo add argo https://argoproj.github.io/argo-helm
   helm install argocd argo/argo-cd -n argocd --create-namespace
   ```
3. Wait a bit, you should see the Argo pods. Proceed once they are all *Running*.
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

Now 

## Quick portforward copy paste


```
kubectl port-forward service/argocd-server -n argocd 8080:443&
kubectl port-forward svc/monitoring-grafana -n monitoring 8081:80&
kubectl port-forward svc/prometheus-operated -n monitoring  9090&
```

**Quick Links**

ArgoCD https://localhost:8080/
Grafana http://localhost:8081/
Prometheus http://localhost:9090/


# Requirements 

## Requirement 1 ArgoCD managing its own lifecycle 

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

## Requirement 2 Prometheus works with ArgoCD metrics and has an alert, plus Grafana

Prometheus UI can be viewed with a port forward

*kubectl port-forward svc/prometheus-operated -n monitoring  9090*

Then visit http://localhost:9090 and take a look, queries for argocd metrics like argocd_app_info should work. And two custom alerts were added named  ArgoCDAppOutOfSync and ArgoCDAppDegraded

For grafana, forward a port as you please, such as 

kubectl port-forward svc/monitoring-grafana -n monitoring 8081:80

Then visit http://localhost:8081/ , the default user is admin, the password is in a secret. Retrieve it via:

```
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" -n monitoring | base64 --decode ; echo
```

An ArgoCD dashboard can be found with working metrics/data.

## Requirement 3 Helm binary changed/updated in repo-server pod

Default for the ArgoCD repo server container here is:

```
kubectl exec -n argocd deploy/argocd-repo-server -- helm version
Defaulted container "repo-server" out of: repo-server, copyutil (init)
version.BuildInfo{Version:"v3.19.4", GitCommit:"7cfb6e486dac026202556836bb910c37d847793e", GitTreeState:"clean", GoVersion:"go1.24.11"}
```

But by adding an init container to get a new version and mount it, we can update it without having to use a custom image + docker registry. On initial launch you will get this version. 

```
kubectl exec -n argocd deploy/argocd-repo-server -- helm version
Defaulted container "repo-server" out of: repo-server, copyutil (init), download-helm (init)
version.BuildInfo{Version:"v3.20.2", GitCommit:"8fb76d6ab555577e98e23b7500009537a471feee", GitTreeState:"clean", GoVersion:"go1.25.9"}
```

# Troubleshooting

If you match my versions of everything is should work with minikube. To me that is the most likely source of issues is using different systems entirely. However, this project seems simple enough it should work in most k8s clusters.  Not accounting for any network restrictions.

Versions used:

WSL2 / Ubuntu 22.04.5 LTS
* helm 3.8.2
* argocd cli 3.4.3
* minikube 1.38.1
* k8s 1.35.1

# Random notes

I have not used ArgoCD prior to this, and had lots of help combing through docs, examples, and AI tooling.  I have left the commit history in place if you want to review it, though there are no useful git commit messages. I considered flattening it but the thought process is more fun. Hopefully everything works as expected. 




