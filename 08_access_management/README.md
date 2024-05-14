# Gravitee Access Management on Minikube

## Prerequisites 

**AM**

* Create an security domain in AM
* Create an app of type WebApp
* Choose a client ID and secret and copy them (or let AM generated them)
* **CHANGEME** Set the redirect URI to point to the APIM login page (on my environment this is https://apim.example.com/console/)
* Associate the default identity provider with the app
* in the app's settings > OAuth 2.0 / OIDC > Scopes, add openid, profile, and email scopes. This will allow APIM to request information about the user loging in
* If you haven't done so, customize the password policy in AM > Settings > Password Policy. This policy will be applied when creating passwords for users.
* Create a new user. Make sure the user is activated. 

**APIM**

* In APIM > Organization > Authentication, create a new identity provider and choose Gravitee AM as the provider type. Specify the client id and client secret, the Server URL should point to an AM Gateway's URL, for me it is `https://am.gateway.master.gravitee.dev`, set your security domain, and add the scopes openid, profile, and email.
* TODO? Role and Group mappings? 

App: Gravitee APIM on Minikube

```sh
helm repo add graviteeio https://helm.gravitee.io
```

```sh
helm upgrade --install --set license.key=${GRAVITEESOURCE_LICENSE_B64} graviteeio-am graviteeio/am -f values.yaml --create-namespace --namespace gravitee-am
```



```sh

```



```sh

```


```sh

```


```sh

```
