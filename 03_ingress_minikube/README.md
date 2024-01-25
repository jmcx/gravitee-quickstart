# Gravitee Ingress Controller on minikube

Change into the folder for this step of the tutorial:

```sh
cd ../03_ingress_minikube
```

Lets use Gravitee as the Ingress gateway and controller for our API calls, instead of minikube and NGINX which we've been using up till now (with `minikube addons enable ingress`). 

However we'll keep minikube ingress running for the UI. 

Now we update our values.yaml to include a parameter that indicates to Gravitee *gateway* not to use another ingress from the cluster (`ingress:enabled:false`). However this won't stop use from using NGINX ingress for the UI. We could also use Gravitee ingress for the UI, but it requires some extra config that we won't bother getting into now. 

We're also configuring the `service` that Gravitee gateway will be exposed as, setting its type to `NodePort`. 

```yaml
gateway:
  # Removing this instruction which is not needed because we aren't exposing the gateway through NGINX ingress anymore
  # ingress:
  #   annotations:
  #     nginx.ingress.kubernetes.io/rewrite-target: /$1
  #   path: /gateway/?(.*)
  services:
    sync:
      kubernetes:
        enabled: true
  ingress:
    enabled: false
  service:
    type: NodePort
    externalPort: 443
```

We'll apply this new values.yml file:

```sh
helm upgrade --set license.key=${GRAVITEESOURCE_LICENSE_B64} --install gravitee-apim graviteeio/apim -f 03_values.yml --namespace gravitee
```

After doing this, if you're following along from the previous guide using GKO, you'll no longer be able to reach your imperative API:

```sh
curl apim.example.com/gateway/echo-imperative                                                                                                      
```

You might receive some html coming from another Gravitee app (probably the portal UI).

This is because the NGINX ingress is no longer configured for the gateway. 

## Create an ingress

Lets now expose the Echo service through our Gravitee Ingress gateway. 

```sh
kubectl apply -f echo-ingress.yml 
```

To make the Gravitee gateway accessible to the outside world, we'll use the minikube service command:

```sh
% minikube service gravitee-apim-gateway --url -n gravitee
http://127.0.0.1:53310
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

The port number show above will be different for you. 

Now we can hit the /echo path from the Ingress resource:

```sh
curl http://127.0.0.1:53310/echo   
{
  "headers" : {
    "Accept" : "*/*",
    "Host" : "gravitee-echo-api.gravitee.svc.cluster.local",
    "User-Agent" : "curl/8.1.2",
    "X-Gravitee-Request-Id" : "c45f89ba-fa26-47c3-9f89-bafa26c7c371",
    "X-Gravitee-Transaction-Id" : "c45f89ba-fa26-47c3-9f89-bafa26c7c371",
    "accept-encoding" : "deflate, gzip"
  }
}
```

So we're still hitting the same echo service, but through a route manage by Gravitee as an ingress controller. 

## Do some API management on the ingress



