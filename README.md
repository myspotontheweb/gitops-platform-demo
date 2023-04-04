# gitops-platform-demo

This is a demo repo to showcase how to manage the standard services running on a k8s platform

# Setup

## Software installation

Setup [Arkade](https://arkade.dev)

```bash
curl -sLS https://get.arkade.dev | sudo sh
echo 'PATH=$PATH:~/.arkade/bin' >> ~/.bashrc
```

Install tools needed for demo

```bash
ark get kubectl kubectx kubens helm yq jq k3d argocd
```

## Cluster installation 

Start a k3d cluster

```bash
k3d cluster create maincluster --k3s-arg "--disable=traefik@server:0"
```

Perform a core install of ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

# Usage

Login

```bash
kubectl config set-context --current --namespace argocd
argocd login --core
```

Create a bootstrap application to install the platform services. Note the path setting will install services for the dev cluster

```bash
argocd app create platform-bootstrap \
   --repo https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path argocd \
   --dest-server https://kubernetes.default.svc \
   --dest-namespace argocd

argocd app set platform-bootstrap --sync-policy automated --self-heal
```

Run the dashboard locally

```bash
argocd admin dashboard
```

Argo CD UI is available at http://localhost:8080

## Add a second cluster to ArgoCD

Add the following entry to the /etc/hosts file

```bash
0.0.0.0 host.k3d.internal
```

Create a second k3d cluster

```bash
k3d cluster create secondcluster --k3s-arg "--tls-san=host.k3d.internal@server:0" --k3s-arg "--disable=traefik@server:0" --kubeconfig-switch-context=false
```

Add this to ArgoCD

```bash
k3d kubeconfig get secondcluster | sed 's/0.0.0.0/host.k3d.internal/' | tee secondcluster.yaml
argocd cluster add k3d-secondcluster --kubeconfig secondcluster.yaml --yes
```

Observe how applications are automatically installed onto the second cluster

# Configuration

Each cluster application is configure using helm

```bash
apps
└── ingress-nginx
    └── Chart.yaml
```

Each helm chart specifies what should be installed using dependencies as follows:

```yaml
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

```bash
#
# Download chart dependencies
#
helm dependency build ./apps/ingress-nginx

#
# Generate the YAML
#
helm template test1 ./apps/ingress-nginx | yq .
```

