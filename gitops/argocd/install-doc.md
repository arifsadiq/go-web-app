# Argo CD Installation

## Install Argo CD using manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Edit the service 'argocd-server' and change the type from ClusterIP to LoadBalancer

```bash
kubectl edit service/argocd-server -n argocd
```

## To get the Load balancer IP

```bash
kubectl get svc argocd-server -n argocd
```