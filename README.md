# gitops-platform-demo
This is a demo repo to showcase how to manage the standard services running on a k8s platform

## Usage

Create a bootstrap application to install the platform services. Note the path setting will install services for the dev cluster

```
argocd app create platform-bootstrap \
   --repo https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path environments/dev \
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
        ├── staging
        │   ├── Chart.yaml
        │   └── ..
        └── prod
            ├── Chart.yaml
            └── ..
```

or alternatively if the same chart is to be installed on all environments

```
├── addons
    └── ingress-nginx
        └── common
            ├── Chart.yaml
            └── ..
```

### Keeping configuration DRY

This is typically what how an addon is configured for a particular environment

```
├── addons
    └── ingress-nginx
        ├── dev
            ├── Chart.yaml
            ├── Chart.lock
            └── values.yaml
```

Instead of defining templates the functional helm chart is declared as a dependency, in the file [Chart.yaml](https://github.com/myspotontheweb/gitops-platform-demo/blob/main/addons/ingress-nginx/common/Chart.yaml)
This enanbles us to control which version of the chart is running on the target environment.

```
apiVersion: v2
name: ingress-nginx
description: Install the nginx ingress controller
type: application
version: 4.3.0
appVersion: "1.4.0"

dependencies:
  - name: ingress-nginx
    repository: https://kubernetes.github.io/ingress-nginx
    version: 4.3.0
```

The Chart.lock file contains the checksum for the downloaded chart, as a security measure. 
Lastly the values.yaml file contains any chart value overrides for this environment install.

### Testing the helm charts

A helm chart can be tested as follows

```
#
# Download chart dependencies
#
helm dependency build ./addons/ingress-nginx/common

#
# Generate the YAML
#
helm template test1 ./addons/ingress-nginx/common | yq .
```

### ArgoCD integration

It's possible to support different environments by creating using one of the following directories, containing two ArgoCD files.w
One to create the encapsulating [Project](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#projects)
and the other to generate applications, the [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/)

```
└── environments
    ├── dev
    │   ├── applicationset.yaml
    │   └── project.yaml
    ├── staging
    │   ├── applicationset.yaml
    │   └── project.yaml
    └── prod
        ├── applicationset.yaml
        └── project.yaml
```

Services are installed into a target instance of ArgoCD by creating a bootstrap application, with the "path" set appropriately:

```
argocd app create platform-bootstrap \
   --repo https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path environments/prod \
   ..
```
