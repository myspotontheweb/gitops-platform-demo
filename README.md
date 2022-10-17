# gitops-platform-demo
This is a demo repo to showcase how to manage the standard services running on a k8s platform

## Usage

Create a bootstrap application to install the platform services. Note that path is installing the "dev" services

```
argocd app create platform-bootstrap \
   --repo  https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path projects/dev \
   --dest-server https://kubernetes.default.svc \
   --dest-namespace argocd

argocd app set platform-bootstrap --sync-policy automated --self-heal
```

## Configuration

Each cluster addon is configured as follows. Each addon specifies 3 charts, one for each type of cluster.

```
├── addons
    └── ingress-nginx
        ├── dev
        │   ├── Chart.yaml
        │   └── ..
        ├── nonprod
        │   ├── Chart.yaml
        │   └── ..
        └── prod
            ├── Chart.yaml
            └── ..
```

### Testing the helm charts

A helm chart can be tested as follows

```
#
# Download chart dependencies
#
helm dependency build ./addons/ingress-nginx/dev

#
# Generate the YAML
#
helm template test1 ./addons/ingress-nginx/dev | yq .
```

### ArgoCD integration

It's possible to support different categories of cluster by installing using one of the following directories, containing two ArgoCD files. One to create the encapsulating [Project](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#projects) and the other to generate applications, the [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/)

```
└── projects
    ├── dev
    │   ├── applicationset.yaml
    │   └── project.yaml
    ├── nonprod
    │   ├── applicationset.yaml
    │   └── project.yaml
    └── prod
        ├── applicationset.yaml
        └── project.yaml
```

Services are installed into a target instance of ArgoCD by creating a bootstrap application, with the "path" set appropriately:

```
argocd app create platform-bootstrap \
   --repo  https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path projects/prod \
   ..
```
