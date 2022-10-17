# gitops-platform-demo
This is a demo repo to showcase how to manage the standard services running on a k8s platform

## Usage

Create a bootstrap application to install the platform services. 

```
argocd app create platform-bootstrap \
   --repo  https://github.com/myspotontheweb/gitops-platform-demo.git \
   --path projects/dev \
   --dest-server https://kubernetes.default.svc \
   --dest-namespace argocd

argocd app set platform-bootstrap --sync-policy automated --self-heal
```

