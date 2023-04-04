# gitops-platform-demo

This is a demo repo to showcase how to manage the standard services running on a k8s platform

# Setup

## Software installation

Setup [Arkade](https://arkade.dev)

```
#
# Download script
#
curl -sLS https://get.arkade.dev | sudo sh

#
# Update path
#
echo 'PATH=$PATH:~/.arkade/bin' >> ~/.bashrc
```

Install tools needed for demo

```
ark get kubectl kubectx kubens helm yq jq minikube argocd
```

## Cluster installation 

Start a minikube cluster

```
minikube start --driver=kvm2
```

Perform a core install of ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

# Usage

Login

```
kubectl config set-context --current --namespace argocd
argocd login --core
```

Create a bootstrap application to install the platform services. Note the path setting will install services for the dev cluster

```
argocd app create platform-bootstrap \
   --repo https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path argocd \
   --dest-server https://kubernetes.default.svc \
   --dest-namespace argocd

argocd app set platform-bootstrap --sync-policy automated --self-heal
```

Run the dashboard locally

```
argocd admin dashboard
```

Argo CD UI is available at http://localhost:8080


# Configuration

Each cluster application is configure using helm

```
apps
└── ingress-nginx
    └── Chart.yaml
```

Each helm chart specifies what should be installed using dependencies as follows:

```
apiVersion: v2
name: ingress-nginx
description: Install the nginx ingress controller
type: application
version: 4.6.0
appVersion: "1.7.0"

dependencies:
  - name: ingress-nginx
    repository: https://kubernetes.github.io/ingress-nginx
    version: 4.6.0
```


## Testing the helm charts

A helm chart can be tested as follows

```
#
# Download chart dependencies
#
helm dependency build ./apps/ingress-nginx

#
# Generate the YAML
#
helm template test1 ./apps/ingress-nginx | yq .
```

