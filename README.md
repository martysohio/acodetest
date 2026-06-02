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
    You will see a new app appear in ArgoCD
   
