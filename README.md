# README

Code for [Rails 6 With Docker, Production Ready!](https://joel-azemar.medium.com/rails-6-with-docker-production-ready-840a88bdaab9)

# Start

Create the network

```
docker network create weblog-bridge-docker-network
```

Create data volume

```
docker volume create weblog-db-data
```

Start database

```
docker run --rm --detach \
  --name weblog-db \
  --env POSTGRES_PASSWORD=postgres \
  --env POSTGRES_USER=postgres \
  --network weblog-bridge-docker-network \
  --mount source=weblog-db-data,target=/var/lib/postgresql/data \
  postgres:latest
```

Create data volume

```
docker volume create weblog-redis-data
```

Start redis database

```
docker run --rm --detach \
 --name weblog-redis \
 --network weblog-bridge-docker-network \
 --mount source=weblog-redis-data,target=/var/lib/redis/data \
 redis:latest
```

build image

```
docker build --squash \
  --tag joel/weblog:latest \
  .
```

start container

```
docker run --rm \
  --name weblogprod-app \
  --env RAILS_LOG_TO_STDOUT=true \
  --env RAILS_MAX_THREADS=8 \
  --env RAILS_MIN_THREADS=1 \
  --env WEB_CONCURRENCY=1 \
  --env REDIS_URL=redis://weblog-redis:6379/1 \
  --env DATABASE_URL="postgres://postgres:postgres@weblog-db:5432/weblog_db_prod?pool=5" \
  --network weblog-bridge-docker-network \
  --publish 3025:3000 \
  -it joel/weblog:latest sh
```

Create database

```
docker exec weblog_prod_app sh -c 'rails db:create db:migrate'
```

Start app

```
docker run --rm \
  --name weblogprod-app \
  --env RAILS_LOG_TO_STDOUT=true \
  --env RAILS_MAX_THREADS=8 \
  --env RAILS_MIN_THREADS=1 \
  --env WEB_CONCURRENCY=1 \
  --env REDIS_URL=redis://weblog-redis:6379/1 \
  --env DATABASE_URL="postgres://postgres:postgres@weblog-db:5432/weblog_db_prod?pool=5" \
  --network weblog-bridge-docker-network \
  --publish 3025:3000 \
  joel/weblog:latest rails server -p 3000 --early-hints -b 0.0.0.0
```