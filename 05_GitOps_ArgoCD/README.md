# GitOps for API management with Gravitee and ArgoCD

GitOps for API management, sometimes known as APIOps, refers to the use of sofware development practices to implement your company's API management strategy. This way, you can expect increased levels of automation, reliability, and scale. 

This approach is particularly well suited to being impemented in conjunction with Kubernetes. Key parts of this approach are:

* using structured text files (i.e. code) to define the desired state of the system declaratively
* using source control systems like git to version and collaborated on those files
* using CI/CD automation to ensure the latest changes to version control are reflected in the API platform.

The below diagram illustrates an example implementation of GitOps for API management with Gravitee. Imagine an organization with multiple teams of API publishers, that are all sharing a common infrastructure but want isolation across teams. Each team would have their own git repository for storing and collaborating on their API definitions. These take the form of Gravitee Custom Resource definitions for Kubernetes. API definitions are automatically deployed to Kubernetes using ArgoCD. On the cluster, dedicated namespaces have been created for each team, and each namespace is enabled with the Gravitee stack which includes:

* the Gravitee Kubernetes Operator (GKO), which listens for Gravitee custom resources like API definitions
* instances of the Gravitee API Gateway, whose configuration is provided to it by the Gravitee Kubernetes Operator

![GitOps with Gravitee](assets/gitopswithgravitee.png "GitOps with Gravitee")

Let's take a look step by step at what is required to implement a simplified version of this. It should give you a feel for the power and elegance of this approach.

# Setting up

There are a few prerequisites to getting this example working:

* a Kubernetes cluster, kubectl
* GKO is intalled on the cluster
* Access to Gravitee platform (either in the cluster, in the cloud, anywhere really!)

In this example, we'll use ArgoCD via its CLI, without needing to open the user interface. This is the simplest setup to get started quickly. Later if you're interested you can also setup the ArgoCD UI. For more on getting started with ArgoCD, head to the official ArgoCD [getting started guide](https://argo-cd.readthedocs.io/en/stable/getting_started/).

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

We'll install the ArgoCD CLI to interact more easily with Argo on our cluster. We'll also login to the ArgoCD installation with the CLI. 

```sh
brew install argocd
argocd login --core
```

Now we can start building ArgoCD applications, which are the unit of deployment that ArgoCD will track. An application points to a repository that contains the Kubernetes manifests for the resources that we want to deploy. In this case, we'll point to https://github.com/jmcx/gravitee-quickstart.git that contains some example API CRDs that we can deploy. 

```sh
argocd app create graviteegitopsapis --repo https://github.com/jmcx/gravitee-quickstart.git --path 05_GitOps_ArgoCD/resources --dest-server https://kubernetes.default.svc --dest-namespace default
```

```sh
argocd app create graviteesampleapicrds --repo https://github.com/gravitee-io/gravitee-kubernetes-operator --path examples/apim/api_definition --dest-server https://kubernetes.default.svc --dest-namespace default
```


```sh
% argocd app get graviteegitopsapis 
Name:               argocd/graviteegitopsapis
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                http://localhost:51317/applications/graviteegitopsapis
Repo:               https://github.com/jmcx/gravitee-quickstart.git
Target:             
Path:               05_GitOps_ArgoCD/resources
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (f875b0b)
Health Status:      Healthy

GROUP        KIND           NAMESPACE  NAME             STATUS     HEALTH   HOOK  MESSAGE
gravitee.io  ApiDefinition  gravitee   echo-api-argocd  OutOfSync  Missing   
```

```sh
% argocd app sync graviteegitopsapis
TIMESTAMP                  GROUP              KIND      NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-04-18T22:44:27+02:00  gravitee.io  ApiDefinition    gravitee       echo-api-argocd  OutOfSync  Missing              
2024-04-18T22:44:27+02:00  gravitee.io  ApiDefinition    gravitee       echo-api-argocd  OutOfSync  Missing              apidefinition.gravitee.io/echo-api-argocd created
2024-04-18T22:44:27+02:00  gravitee.io  ApiDefinition    gravitee       echo-api-argocd    Synced  Missing              apidefinition.gravitee.io/echo-api-argocd created

Name:               argocd/graviteegitopsapis
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                http://localhost:51365/applications/graviteegitopsapis
Repo:               https://github.com/jmcx/gravitee-quickstart.git
Target:             
Path:               05_GitOps_ArgoCD/resources
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (f875b0b)
Health Status:      Healthy

Operation:          Sync
Sync Revision:      f875b0b9d61002f95061e04cce72265cab4f6212
Phase:              Succeeded
Start:              2024-04-18 22:44:27 +0200 CEST
Finished:           2024-04-18 22:44:27 +0200 CEST
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP        KIND           NAMESPACE  NAME             STATUS  HEALTH  HOOK  MESSAGE
gravitee.io  ApiDefinition  gravitee   echo-api-argocd  Synced                apidefinition.gravitee.io/echo-api-argocd created
```

```sh
% curl apim.example.com/gateway/echo-argocd     
{
  "headers" : {
    "Accept" : "*/*",
    "Host" : "gravitee-echo-api.gravitee.svc.cluster.local",
    "User-Agent" : "curl/8.4.0",
    "X-Forwarded-For" : "10.244.0.1",
    "X-Forwarded-Host" : "apim.example.com",
    "X-Forwarded-Port" : "80",
    "X-Forwarded-Proto" : "http",
    "X-Forwarded-Scheme" : "http",
    "X-Gravitee-Request-Id" : "1d586f5a-3164-4170-986f-5a31646170dd",
    "X-Gravitee-Transaction-Id" : "1d586f5a-3164-4170-986f-5a31646170dd",
    "X-Real-IP" : "10.244.0.1",
    "X-Request-ID" : "18e12498daada3b3087a4fbd102314bc",
    "X-Scheme" : "http",
    "accept-encoding" : "deflate, gzip"
  }
}
```