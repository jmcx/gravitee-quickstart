# Kustomize

Kustomize can either be used from the command line if the CLI is installed, e.g.: `kustomize build ./base | kubectl apply -f -`.

But ideally, if kubernetes version is higher than 1.14, then it can be use directly from `kubectl`: 

* `kubectl kustomize [dir]` will show you a preview of the customized yaml manifests
* `kubectl apply -k [dir]` will create the resources

This repo contains three API defintions:

* Movies API
* Books API
* Music API

These all have a predefined keyless plan called "KEY_LESS", and each API has a specific label, "Movies", "Books", and "Music" respectively. 

The kustomize file in the base directory will apply a patch defined in `set-labels.yaml` that will add a label called "Entertainment" to all three APIs.

You can test the output of this by running the following command:

```sh
% k kustomize base               
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: books-api
spec:
  contextRef:
    name: dev-ctx
    namespace: gravitee
  description: An API for books
  labels:
  - Books
  - Entertainment
  name: books-api
  plans:
  - description: FREE
    name: KEY_LESS
    security: KEY_LESS
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

Next, using Kustomize's overlays method, we've created one directory per overaly, each representing an environmnet: development and production. 

In each directory, a `kustomization.yaml` file points to the patches to be applied on that environment, in each case to a dedicated `set-plan.yaml` patch. The one for development sets the API's plans array to a new keyless plan, the one for production set the API's plans array to a new API key plan.

These patches completely replace the value of the plans array.

We can now see that by applying the development overlay, we get APIs with keyless plans:

```sh
% k kustomize overlays/development 
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: books-api
spec:
  contextRef:
    name: dev-ctx
    namespace: gravitee
  description: An API for books
  labels:
  - Books
  - Entertainment
  name: books-api
  plans:
  - description: Common keyless for all production APIs
    name: Common keyless plan
    security: KEYLESS
    status: PUBLISHED
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

and in production we get API key plans:

```sh
% k kustomize overlays/production 
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: books-api
spec:
  contextRef:
    name: dev-ctx
    namespace: gravitee
  description: An API for books
  labels:
  - Books
  - Entertainment
  name: books-api
  plans:
  - description: Common API key plan for all production APIs
    name: Common API key plan
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

Plus, the common patches from the base directory are also applied! This is because the kustomization files for each overlay refer to the base directory.

To apply one of the for real, you just need to run the following command:

```sh
% k apply -k overlays/production
apidefinition.gravitee.io/books-api created
apidefinition.gravitee.io/movies-api created
apidefinition.gravitee.io/music-api created
```

The new APIs should all show the labels and plans as expected.

You can delete these easily with the reverse operation:

```sh
% k delete -k overlays/production
apidefinition.gravitee.io "books-api" deleted
apidefinition.gravitee.io "movies-api" deleted
apidefinition.gravitee.io "music-api" deleted
```

This shows how easily you can externalise common aspects of API definitions such as labels and plans, and apply them to many API definitions, while using overlays to manage things like environments. 

## Future improvements

### ArgoCD

I'll soon show how you can do this with ArgoCD.

### Categories

When category support for v2 APIs is released (coming very soon), we will be able to apply categories with Kustomize. For example, by adding the following snippet to base/kustomization.yaml:

```sh
- path: set-category.yaml
  target: 
    kind: ApiDefinition
```

And creating a patch file called `set-category.yaml` in the `base` directory with the following contents:

```sh
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: required-for-kustomize-but-not-used
spec:
  categories: ["Entertainement"]
```

Because this is using the so-called strategic merge method of Kustomize, this will add a single category to each API definition, replacing any existing ones. For an incremental approach that adds new categories, see the JSON Patch method used in `set-label.yaml` patch. 