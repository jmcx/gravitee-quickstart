# Kubernetes Gateway API 

Until this is supported by Gravitee (it is on the roadmap), this example uses NGINX. 

Most of this example is taken from NGINX gateway docs:
* [Install](https://docs.nginx.com/nginx-gateway-fabric/installation/installing-ngf/helm/)
* [Expose](https://docs.nginx.com/nginx-gateway-fabric/installation/expose-nginx-gateway-fabric/)
* [Example app](https://docs.nginx.com/nginx-gateway-fabric/how-to/traffic-management/routing-traffic-to-your-app/)

```sh
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
```

We'll install with service type nodeport to make it easier to get this working locally with Minikube

```sh
helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway
```

Create an example app:

```sh
kubectl apply -f coffee-app.yaml
```

Create the Gateway API resources. First the Gateway:

```sh
kubectl apply -f gateway.yaml
```

Then the HTTP Route:

```sh
kubectl apply -f http-route.yaml
```

Expose the NGINX gateway service to the outside of Minikube:

```sh
minikube service ngf-nginx-gateway-fabric -n nginx-gateway
|---------------|--------------------------|-------------|---------------------------|
|   NAMESPACE   |           NAME           | TARGET PORT |            URL            |
|---------------|--------------------------|-------------|---------------------------|
| nginx-gateway | ngf-nginx-gateway-fabric | http/80     | http://192.168.49.2:30492 |
|               |                          | https/443   | http://192.168.49.2:30613 |
|---------------|--------------------------|-------------|---------------------------|
🏃  Starting tunnel for service ngf-nginx-gateway-fabric.
|---------------|--------------------------|-------------|------------------------|
|   NAMESPACE   |           NAME           | TARGET PORT |          URL           |
|---------------|--------------------------|-------------|------------------------|
| nginx-gateway | ngf-nginx-gateway-fabric |             | http://127.0.0.1:58931 |
|               |                          |             | http://127.0.0.1:58932 |
|---------------|--------------------------|-------------|------------------------|
[nginx-gateway ngf-nginx-gateway-fabric  http://127.0.0.1:58931
http://127.0.0.1:58932]
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

Test the configuration:

```sh
curl http://127.0.0.1:58931 -H "host: cafe.example.com"
Server address: 10.244.0.40:8080
Server name: coffee-6b8b6d6486-b4ccc
Date: 13/May/2024:13:03:24 +0000
URI: /
Request ID: 6c3e82f70660bf2f014b19cc016c4aa7
```