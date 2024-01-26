# Gravitee on minikube

Change into the dedicated folder for this part of the tutorial:

```sh
cd 01_gravitee_minikube
```

Start minikube with some extra resources:

```sh
minikube start --memory 4096 --cpus 4
```

Enable minikube ingress:

```sh
minikube addons enable ingress
```

The ingress addon will add the NGINX ingress controller, which will be used to expose Gravitee components (UI, mAPI, gateway) outside of the K8s cluster, so that we can open the UI in our browser and make requests to the gateway.

Run minikube tunnel in a separate terminal (it will block that terminal). Because minikube is running in a virtual machine, tunnel will help expose the ingress resources to the host machine's network (your local machine). In real life, on a real cluster, this would work differently. Because it will modify network configs, it requires root permissions. Either run it with sudo, or you'll need to come back to it later to give it your password (and if you forget, you might not understand why things aren't working):

```sh
sudo minikube tunnel
```

## Install a light setup of Gravitee on K8s 

The approach here is based on a simplified version of the Helm values file `value-light.yml` which can be found in the Gravitee official docs. Simplified in that we make mongo and elastic run as single node clusters, as opposed to having 3 nodes for each.

> !!! If running a version of NGINX ingress > 1.9.0, there is an issue with an annotation that we use (`nginx.ingress.kubernetes.io/configuration-snippet: "etag on;\nproxy_pass_header ETag;\nproxy_pass_header if-match;\n"`) that is now forbidden by default by NGINX (see https://github.com/kubernetes/ingress-nginx/issues/10452). 
>
> To check which version of NGINX is run by minikube, the easiest way is to check the image version printed in the terminal output after you ran `minikube addons enable ingress`. Otherwise you can fish around with a command similar to `kubectl get deployment ingress-nginx-controller -o yaml -n ingress-nginx | grep -i image`, assuming here that the ingress controller's namespace is `ingress-nginx`.
>
>To revert this change in NGINX and allow the annotations, edit the nginx controller's config map and add new line `allow-snippet-annotations: "true"` to the data section as shown here (again assuming here that the ingress controller's namespace is `ingress-nginx`). 
>
> If your default edit is `vi`, then enter `i` to enter edit mode, paste the annotation in the right place, hit `ESC` to leave edit mode, and hit `:wq` followed by `ENTER` to save and exit. 
>
> ```sh
> kubectl edit cm ingress-nginx-controller -n ingress-nginx
> ```
>
> ```sh
> apiVersion: v1
> data:
>   allow-snippet-annotations: "true"
>   hsts: "false"
> kind: ConfigMap
> metadata:
> ...
> ```

Now add the Gravitee Helm chart repository:

```sh
helm repo add graviteeio https://helm.gravitee.io
```

If you want to use the enterprise version of the product, following the instructions below to add your enterprise license as a Helm argument:

> **ADDING A LICENSE** 
>
> *RECOMMENDED:* You can pass the license to Helm as an argument: `--set license.key=<license.key in base64>`
>
> To get the license.key value, encode your file license.key in base64:
>
> macOS: base64 license/license.key
>
> linux: base64 -w 0 license/license.key
>
> Two example equivalent commands for macOS:
>
> `export GRAVITEESOURCE_LICENSE_B64="$(cat license.key | base64)"`
> `export GRAVITEESOURCE_LICENSE_B64="$(base64 -i license.key)"`
>
> *ALTERNATIVE:* You can also reference your license directly in the values.yaml file, at the root as shown below. Less advisable as you might want to push this values.yaml to version control somewhere.
>
> ```yaml
> license: 
>  name: gravitee-ee-license
>  key: ...
> ```
>
> Again, if you need to convert a license key from base64 into text for the values.yaml, you can do `cat license.key | base64`. 
>

Now we can install the Gravitee Helm chart, here passing the license key as an argument as explained above:

```sh
helm install --set license.key=${GRAVITEESOURCE_LICENSE_B64} gravitee-apim graviteeio/apim -f 01_values.yml --create-namespace --namespace gravitee
```

As suggested, you can now run a command to watch as the pods start up, this can take a couple of minutes:

```sh
kubectl get pods --namespace=gravitee -l app.kubernetes.io/instance=gravitee-apim -w
NAME                                                 READY   STATUS    RESTARTS   AGE
gravitee-apim-api-fc4b6c9d7-b4blc                    0/1     Running   0          88s
gravitee-apim-gateway-7b4f96747-7nw6s                1/1     Running   0          88s
gravitee-apim-portal-786694bd44-vjh6p                1/1     Running   0          88s
gravitee-apim-ui-9d7c95f8b-hmdx2                     1/1     Running   0          88s
graviteeio-apim-elasticsearch-master-0               0/1     Running   0          88s
graviteeio-apim-mongodb-replicaset-bf444d6c7-np5qw   1/1     Running   0          88s
```

If later on you make changes to the values.yaml file you can upgrade the installation using the following command:

```sh
helm upgrade --set license.key=${GRAVITEESOURCE_LICENSE_B64} --install gravitee-apim graviteeio/apim -f 01_values.yml --namespace gravitee
```

And you can comletely delete the installation with:

```sh
helm delete gravitee-apim
```

You need to add a new entry to your `/etc/hosts` file, because by [default the Gravitee Helm chart](https://github.com/gravitee-io/gravitee-api-management/blob/master/helm/values.yaml) will set host `apim.example.com` for the installation:

```sh
127.0.0.1     apim.example.com
```

Once all the pods have started, you should be able to open the applications (you may need to accept the self-signed certificates):

Console: https://apim.example.com/console

Portal: https://apim.example.com/catalog/

user/pass: `admin/admin`

## Deploy an API

Now that everything is running, we can deploy an API and proxy it with the gateway.

For a more complete example, lets deploy a simple service on the cluster call the "echo API". This will run a microservice in your cluster, that exposes a K8s service. When invoked, this service responds back to the caller with metadata about the HTTP call. Install it like so:

```sh
kubectl apply -f gravitee-echo-api.yml
```

We'll now create an API in Gravitee to expose this service through the Gravitee gateway. An API definition file called `Echo-API-imperative-1.json` (imperative as opposed to the declarative API we'll create with GKO later, and `1` means version 1) is provided that you can import into the Gravitee console to create the necessary API. To do this, open the Gravitee console in the browser, head to *APIs*, click on *+ Add API*, select *Import an API definition*, and upload the file. 

Make sure the API is deployed, **and also STARTED !!!**. You'll need to click in the UI to do this.

The main characteristics of this Gravitee API are:
* its endpoint refers to the Echo API service running in the cluster at the address `http://gravitee-echo-api.gravitee.svc.cluster.local:80`. 
* its exposes an endpoint on the path `/echo-imperative`

You could also create this API manually in the console if you prefer, rather than import this API definition. 

You should be able to call the API now:

```sh
curl apim.example.com/gateway/echo-imperative
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
    "X-Gravitee-Request-Id" : "89c04cc8-5dc2-446f-804c-c85dc2e46f4e",
    "X-Gravitee-Transaction-Id" : "89c04cc8-5dc2-446f-804c-c85dc2e46f4e",
    "X-Real-IP" : "10.244.0.1",
    "X-Request-ID" : "fb9b6ca18c29470535230fc86c323512",
    "X-Scheme" : "http",
    "accept-encoding" : "deflate, gzip"
  }
}
```

As expected, it spits back a bunch of metadata about the request. 

If it doesn't work and instead you're getting something like this:

```sh
curl apim.example.com/gateway/echo-imperative
No context-path matches the request URI.
```

Then check again to make sure you have **BOTH** deployed **AND** started the API in the UI.