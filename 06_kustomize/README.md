# Kustomize

Running the following command will give you a preview of the patched resources:

```sh
% k kustomize ./
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: books-api
spec:
  contextRef:
    name: dev-ctx
    namespace: gravitee
  description: An API for books
  name: books-api
  plans:
  - description: my transversal API key plan
    name: My API key plan
    security: API_KEY
    status: PUBLISHED
    validation: MANUAL
  proxy:
    groups:
    - endpoints:
      - name: Default
        target: https://api.gravitee.io/echo
    virtual_hosts:
    - path: /books
  version: "1"
  ...
```

As you can see, the keyless plan of each API has been replaced by an API Key plan. 

To apply these to the cluster, you can run:

```sh
% k apply -k ./ 
apidefinition.gravitee.io/books-api created
apidefinition.gravitee.io/movies-api created
apidefinition.gravitee.io/music-api created
```

The APIs created will all have the patched API key plan. 

