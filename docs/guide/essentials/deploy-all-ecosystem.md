# Deploy All Ecosystem

### For all enviroment you should run the follow images:

- ADempiere gRPC: https://hub.docker.com/r/erpya/adempiere-grpc-all-in-one

```shell
docker pull erpya/adempiere-grpc-all-in-one
```

- Proxy ADempiere API: https://hub.docker.com/r/erpya/proxy-adempiere-api

```shell
docker pull erpya/proxy-adempiere-api
```

- ADempiere Vue: https://hub.docker.com/r/erpya/adempiere-vue

```shell
docker pull erpya/adempiere-vue
```

- ADempiere eCommerce: https://hub.docker.com/r/erpya/adempiere-ecommerce

```shell
docker pull erpya/adempiere-ecommerce
```

## Run Docker Stack

```yaml
# docker-compose.yaml
version: '3.7'

services:
  grpc-backend:
    image: erpya/adempiere-grpc-all-in-one
    container_name: adempiere-backend
    stdin_open: true
    tty: true
    environment:
      - SERVER_PORT=50059
      - SERVICES_ENABLED=access; business; core; dashboarding; dictionary; enrollment; log; ui; workflow; store; pos; updater;
      - SERVER_LOG_LEVEL=WARNING
      - DB_HOST=postgres_host
      - DB_PORT=5432
      - DB_NAME=adempiere
      - DB_USER=adempiere
      - DB_PASSWORD=adempiere
      - DB_TYPE=PostgreSQL
    ports:
      - 50059:50059

  redis:
    image: redis:4-alpine
    container_name: adempiere-redis
    stdin_open: true
    tty: true
    ports:
      - '6379:6379'

  es7:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.3.2
    container_name: adempiere-eslastic-search
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
    ports:
      - '9200:9200'
      - '9300:9300'
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xmx512m -Xms512m

  api-rest:
    image: erpya/proxy-adempiere-api
    container_name: adempiere-proxy
    depends_on:
      - es7
      - redis
    stdin_open: true
    tty: true
    environment:
      - SERVER_PORT=8085
      - AD_DEFAULT_HOST=adempiere-backend
      - AD_DEFAULT_PORT=50059
      - ES_HOST=adempiere-eslastic-search
      - ES_PORT=9200
      - VS_ENV=dev
      - INDEX=vue_storefront_catalog
      - RESTORE_DB=N
    ports:
      - 8085:8085

  vue-app:
    image: erpya/adempiere-vue
    container_name: adempiere-frontend
    stdin_open: true
    tty: true
    environment:
      - API_URL=http://adempiere-proxy:8085
    ports:
      - 9526:80

  e-commerce:
    image: erpya/adempiere-ecommerce
    container_name: adempiere-ecommerce
    stdin_open: true
    tty: true
    environment:
      - SERVER_PORT=3000
      - API_URL=http://adempiere-proxy:8085
      - STORE_INDEX=vue_storefront_catalog
      - VS_ENV=dev
    ports:
      - 3000:3000
```

Note: Eslastic Search container requires a config file `elasticsearch.yaml`.

```shell
# requires superuser permissions of the operating system ('su' or 'sudo')
docker-compose up
```

Containers Running:

- adempiere-backend
- adempiere-redis
- adempiere-eslastic-search
- adempiere-proxy
- adempiere-frontend
- adempiere-ecommerce
