# README

Welcome to this repository that contains hands-on guides to get started with different aspects of working with Gravitee on Kubernetes.

## Some tips to make your life easier

### Make sure you have kubectl auto-completion

For example on zsh:

```sh
source <(kubectl completion zsh)
```

Add that command to your ~/.zshrc file (create it if it doesn't exist).

Make sure you also have these two commands early in your `.zshrc` file:

```sh
autoload -Uz compinit
compinit
```

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

### Other helpful commands

If at any point you want to restart a pod during troubleshoot, you can do this by referencing the associated deployment:

```sh
k rollout restart deployment <deployment name>
```
