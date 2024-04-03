# DB-less Gravitee on Minikube

Install a DB-less gateway, that has no dependency on a repository to store its API configurations (like MongoDB), nor a dependency on Elasticsearch for storing logs. 

```sh
helm upgrade --set license.key=${GRAVITEESOURCE_LICENSE_B64} --install gravitee-apim graviteeio/apim -f 04_values.yml --create-namespace --namespace gravitee
```

Now you can create an API, with mode local set to `true` so that the config is stored in a config map in the same namespace as the gateway: 

```sh
kubectl apply -f echo-api-declarative-1.yml
```

You can now call the API:

```sh
curl apim.example.com/gateway/echo-declarative
```