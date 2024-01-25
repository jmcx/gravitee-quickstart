# Gravitee Ingress Controller on minikube

Lets use Gravitee as the Ingress gateway and controller for our API calls, instead of minikube and NGINX which we've been using up till now (with `minikube addons enable ingress`). 

Start by disabling minikube's ingress addon:

```sh
minikube addons disable ingress
```

Now we update our values.yaml to include a parameter that indicates to Gravitee gateway not to use another ingress from the cluster (`ingress:enabled:false`). This means that Gravitee cannot expect to leverage an existing Ingress as part of its configuration.

We're also configuring the `service` that Gravitee gateway will be exposed as, setting its type to `loadbalancer` so that it is accessible from outside the cluster. 

**TODO** I'm not sure yet if the external-dns thing is necessary, I would like to avoid it to make this onboarding easier, so for now I'm not using it. The docs say `"The external-dns.alpha.kubernetes.io/hostname instructs external-dns to use your external DNS provider to create a DNS entry that matches the load balancer service IP."` But I'd rather just use the IP directly for now which is what I'll try to do (and what I think the guide should do too). 

```yaml
gateway:
  services:
    sync:
      kubernetes:
        enabled: true
  ingress:
    enabled: false
  service:
    type: LoadBalancer
    # annotations:
    #   external-dns.alpha.kubernetes.io/hostname: graviteeio.example.com
    externalPort: 443
```

After doing this, if you're following along from the previous guide using GKO, you'll no longer be able to reach your imperative API:

```sh
curl apim.example.com/gateway/echo-imperative                                                                                                      
curl: (56) Recv failure: Connection reset by peer
```

This is because the ingress is no longer configured. To fix this, we'll setup Gravitee as the new ingress gateway.