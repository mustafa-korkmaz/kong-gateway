This repository implements an API Gateway solution and shows the steps how to host 2 services from one entry point using [Kong Gateway](https://docs.konghq.com/gateway/2.8.x/) on Docker. It leverages `deck sync` command in order to apply configurations once Kong Gateway is initialized.

## Features
- [with-a-database](https://docs.konghq.com/gateway/2.8.x/install-and-run/docker/#install-kong-gateway-with-a-database) mode using PostgreSQL
- Uses [decK](https://docs.konghq.com/deck/latest/) in order to sync configuration defined in [`kong.yml`](./declarative/kong.yml)
- Pre-installed and configured [OAuth 2.0 Authentication](https://docs.konghq.com/hub/kong-inc/oauth2/) plugin (optional feature using [`kong-with-oauth2.yml`](./declarative/kong-with-oauth2.yml))
- Easy deploy with `docker compose`

Detailed version for step by step installation in Turkish is [here](https://mustafakorkmaz.site/2022/09/kong-ile-api-gateway-kuruyoruz/).

## Installation 
### 1. Create your user defined docker network
`$ docker network create api_gateway`
### 2. Start services
`$ docker-compose up`

## Service dependencies
![service dependency graph](https://user-images.githubusercontent.com/5656640/189125557-c576c222-9689-43a9-8ac4-4f634e78d069.png)
- **kong_db_migration** depends on **kong_db** which runs a PostgreSQL instance
- **deck** depends on **kong** itself as it needs Kong Gateway to be initialized and healthy
- **kong** awaits **kong_db_migration** service to be completed successfully (exited with code 0)

#### Service creation order
![docker-compose=gif](https://user-images.githubusercontent.com/5656640/189126737-0a1587e4-e4b8-4810-8736-0abde3a1b940.gif)

## Working with ready-to-use OAuth 2.0 Authentication plugin
### 1. Change your declarative file
Use [kong-with-oauth2.yml](./declarative/kong-with-oauth2.yml) file declarations instead of kong.yml in [docker-compose.yml](docker-compose.yml)
### 2. Claim a token with pre-configured credentials
Ready-to-use plugin will be installed for **nginx_service** and configured to work with [client credentials flow](https://www.rfc-editor.org/rfc/rfc6749#section-1.3.4)  
`$ curl -k -X POST https://localhost:8443/nginx_service/oauth2/token?apikey=oauthapikey  
--data "client_id=demo_client"  
--data "client_secret=demo_secret"  
--data "grant_type=client_credentials"`
  
`{"token_type":"bearer","access_token":"Oasov8STaYTlnsvFQoH3GULbrW4VXwdB","expires_in":7200}`

### 2. Pass claimed token as an authorization header value
`$ curl http://localhost:8000/nginx --header "authorization: bearer Oasov8STaYTlnsvFQoH3GULbrW4VXwdB"`  

`<!DOCTYPE html><html>..Welcome to nginx!</h1>..</html>`
