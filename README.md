# acodetest
thing for people

## goals

Deploy Argo CD and prepare a GitOps repo that makes Argo CD do the following:
1. Manage its own configurations/lifecycle
2. Deploy Prometheus and any required configurations to make it monitor Argo CD (with
dashboards and alerts)
3. Replace the Helm binary included with Argo CD with a different version of Helmand specifically use Helm version 3.7
4. Ensure you set a sync wave with a value of -10 on the app-of-apps
Share the git repo with us, and be prepared to discuss what you did and how it works

## k8s setup

* Minikube - test done with 1.26, newer versions should work, but unknown. On WSL 2 Ubuntu 22.04.5 LTS



## Setup steps

1. Initial install of ArgoCD, can do this per Argo's own docs 
https://argo-cd.readthedocs.io/en/stable/getting_started/ and set the context while you are at it.
```
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl config set-context --current --namespace=argocd
```

2. Install ArgoCD cli locally, I used curl/Linux install, others are at https://argo-cd.readthedocs.io/en/stable/cli_installation/
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

3. Reset/retrieve initial password (again same as ArgoCD getting started docs)
```
argocd admin initial-password -n argocd
```

4. Log into the ArgoCD UI, you will need to forward the port if using local/minikube.  
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
