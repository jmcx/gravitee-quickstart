# README

I recommend running through these one by one. There is no info here yet about how to install minikube, kubectl, helm etc... counting on you for that! 

1. [Gravitee on minikube](./01_gravitee_minikube/README.md)
2. [Gravitee Kubernetes Operator (GKO) on minikube](./02_GKO_minikube/README.md)
3. [(WIP) Gravitee Ingress Controller on minikube](./03_ingress_minikube/README.md)
4. [Gravitee and Redpanda event APIs, with Docker](./04_redpanda_docker/README.md)
5. [(WIP) Gravitee on k3d](./05_gravitee_k3d/README.md)

## Pro tips to make your life easier

### Make sure you have kubectl auto-completion

For example on zsh:

```sh
source <(kubectl completion zsh)
```

Add that command to your ~/.zshrc file (create it if it doesn't exist).

You can now hit `TAB` to autocomplete on everything with `kubectl`, can't live without it. 

### Install kubens

This makes it very easy to list the namespaces available: 
```sh
% kubens
default
gravitee
ingress-nginx
kube-node-lease
kube-public
kube-system
```

And change namespace: 

```sh
% kubens default
Context "minikube" modified.
Active namespace is "default".
```

Install it like this: 

```sh
brew install kubectx
```

With brew install, tab-autocompletion should work out of the box. 

