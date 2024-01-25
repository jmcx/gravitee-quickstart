# Gravitee and Redpanda event APIs, with Docker compose 

This approach assumes you have your license key the the location `/license/license.key` from where you are running the `docker compose` commands.

## Start and Stop

```sh
 docker compose up -d
 ```

```sh
 docker compose stop
 ```

## Open the various consoles

Gravitee APIM console: http://localhost:8084/ 

Gravitee Developer Portal: http://localhost:8085/

user/pass: admin/admin.

Redpanda: http://127.0.0.1:8080/

Request bin: https://public.requestbin.com/r/en3qlme6ev0ga/2Yna0HX069KnmJscmymnwTGQi9D

## Create the APIs

### Sync

In the Console UI, create a new API and import file `Corporate-BS-v2-API-1-0.json`.

Deploy it and **START** it!

### Async

In the Console UI, create a new API and import file `Product-orders-V2-API-1-0.json`.

Deploy it and **START** it!

## Consume the REST API

API key plan:

```sh
curl http://localhost:8082/quotev2 -H "X-Gravitee-Api-Key: 390a0b82-85f9-49d7-b50c-b5b00b97c491" 
```

Keyless plan (rate limited):

```sh
curl "localhost:8082/quotev2"
```

## Trigger the event API

```sh 
docker exec -it redpanda-0 rpk cluster info
docker exec -it redpanda-0 rpk topic create orders
docker exec -it redpanda-0 rpk topic produce orders
```

Enter something like:

```json
'{"product": "A Brief History of Time"}'
```

open the console at http://127.0.0.1:8080/topics