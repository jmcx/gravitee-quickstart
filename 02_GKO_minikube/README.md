# Gravitee Kubernetes Operator (GKO) on minikube

## Install GKO

Change into the folder for this step of the tutorial:

```sh
cd ../02_GKO_minikube
```

You'll notice that the `02_values.yml` we use here has a small addition to the `01_values.yml` that we used in the previous step. The added config (shown below) enables GKO to work with the Gravitee gateway, by telling that gateway that it should load API configs from a Kubernetes configMap, instead of from the database:

```yaml
  services:
    sync:
      kubernetes:
        enabled: true
```

If you have already added the Gravitee Helm charts, make sure they are up to date with the following command:

```sh
helm repo update graviteeio
```

So we'll start by updating our installation with this new `values.yml` file:

```sh
helm upgrade --set license.key=${GRAVITEESOURCE_LICENSE_B64} --install gravitee-apim graviteeio/apim -f 02_values.yml --create-namespace --namespace gravitee
```

You should see the gateway pod restart as it updates its configuration.

Now we can install the GKO helm chart:

> !!! There was [an issue](https://gravitee.atlassian.net/browse/APIM-3747) at time of writing requiring the extra `0.11.0` tag shown below in order to make sure we install the latest "stable" release of GKO (in this case, avoiding 1.0.0-x.beta versions). Once this fix is released you won't needed to specify this additional paramater.

```sh
helm install graviteeio-gko graviteeio/gko --namespace gravitee
```

## Create a Management Context

Create the management context. This resource will tell GKO which Gravitee control plane it should connect to, by pointing to the right Gravitee management API we deployed in the previous chapter. What GKO essentially does is translate between declarative `kubectl` commands and calls to this management API:

```sh
kubectl apply -f management-context-with-credentials.yml --namespace gravitee
```

## Create an API

Create an API declaratively. This API also points to the same Echo API service we used previously, but will expose it on a different path which is `/echo-declarative`:

```sh
kubectl apply -f echo-api-declarative-1.yml --namespace gravitee
```

You should now be able to see this API in the Console UI, with a Kubernetes logo next to it signifying that this API is managed by GKO and is shown as read-only in the Console. 

You can now call the API:

```sh
curl apim.example.com/gateway/echo-declarative
{
  "headers" : {
    "Accept" : "*/*",
    "Host" : "gravitee-echo-api.gravitee.svc.cluster.local",
    "User-Agent" : "curl/8.1.2",
    "X-Forwarded-For" : "10.244.0.1",
    "X-Forwarded-Host" : "apim.example.com",
    "X-Forwarded-Port" : "80",
    "X-Forwarded-Proto" : "http",
    "X-Forwarded-Scheme" : "http",
    "X-Gravitee-Request-Id" : "d015d39d-7295-4cf8-95d3-9d7295acf828",
    "X-Gravitee-Transaction-Id" : "d015d39d-7295-4cf8-95d3-9d7295acf828",
    "X-Real-IP" : "10.244.0.1",
    "X-Request-ID" : "946bf760daa0e650cbed1145d99b7b6f",
    "X-Scheme" : "http",
    "accept-encoding" : "deflate, gzip"
  }
}
```

## Create an Application

We can create an Application declaratively. The Application CRD also references the management context, and includes many other options you might want to specify when creating an application:

```sh
kubectl apply -f application.yml --namespace gravitee
```

You should now be able to see this API in the console.