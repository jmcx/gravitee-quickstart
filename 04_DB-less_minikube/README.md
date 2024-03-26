# DB-less Gravitee on Minikube

helm upgrade --set license.key=${GRAVITEESOURCE_LICENSE_B64} --install gravitee-apim graviteeio/apim -f 04_values.yml --create-namespace --namespace gravitee