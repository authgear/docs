---
description: How to run locally with Docker.
---

# Running locally with Docker

Authgear is available as a Docker image. It depends on PostgreSQL (with pg\_partman enabled) and Redis. To run it locally, the simplest way is to use docker-compose.

## Create the project directory

Let's get started with creating a new directory.

```bash
mkdir myapp
cd myapp
```

## Create docker-compose.yaml

The next step is to create `docker-compose.yaml` to setup PostgreSQL, Redis, and Authgear.

You can start with the following `docker-compose.yaml`:

```yaml
version: "3"
services:
  db:
    image: postgres-pg-partman:latest
    build:
      context: ./postgres
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    ports:
      - "5432:5432"

  redis:
    image: redis:6.2.6
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

  authgear:
    # Remember to replace the latest tag with the exact version you would like to use!
    image: quay.io/theauthgear/authgear-server:latest
    volumes:
      - ./authgear.yaml:/app/authgear.yaml
      - ./authgear.secrets.yaml:/app/authgear.secrets.yaml
    environment:
      DEV_MODE: "true"
      LOG_LEVEL: "debug"
    ports:
      - "3000:3000"

volumes:
  redis_data:
    driver: local
  db_data:
    driver: local
```

Note that we need to build the PostgreSQL image ourselves. We can do this with a simple Dockerfile.

```bash
mkdir postgres
touch postgres/Dockerfile
```

Copy the following contents to `postgres/Dockerfile`

```Dockerfile
FROM postgres:12.3

ENV PARTMAN_VERSION 4.5.1

RUN apt-get update && apt-get install -y \
	unzip \
	build-essential \
	postgresql-server-dev-11 \
	wget \
	&& rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/pgpartman/pg_partman/archive/v${PARTMAN_VERSION}.zip -O pg_partman-${PARTMAN_VERSION}.zip && unzip pg_partman-${PARTMAN_VERSION}.zip && cd pg_partman-${PARTMAN_VERSION} && make NO_BGW=1 install
```

## Create authgear.yaml and authgear.secrets.yaml

First, we need to create `authgear.yaml` and `authgear.secrets.yaml`. Authgear itself is a CLI program capable of generating a minimal configuration file.

Run the following command to generate minimal `authgear.yaml` and `authgear.secrets.yaml`:

```bash
docker run --rm -it -w "/work" -v "$PWD:/work" quay.io/theauthgear/authgear-server authgear init
```

This command is interactive and it will prompt you a series of questions. You want to turn off email verification because we do not have SMTP setup. We also need to adjust some endpoints so that Authgear can connect to other services in the network.

```
App ID (default 'my-app'):
HTTP origin of authgear (default 'http://localhost:3000'):
HTTP origin of portal (default 'http://portal.localhost:8000'):
Phone OTP Mode (sms, whatsapp, whatsapp_sms) (default 'sms'):
Would you like to turn off email verification? (In case you don't have SMTP credentials in your initial setup) [Y/N] (default 'false'): Y
Select a service for searching (elasticsearch, postgresql) (default 'elasticsearch'):
Database URL (default 'postgres://postgres:postgres@127.0.0.1:5432/postgres?sslmode=disable'): postgres://postgres:postgres@db:5432/postgres?sslmode=disable
Database schema (default 'public'):
Audit Database URL (default 'postgres://postgres:postgres@127.0.0.1:5432/postgres?sslmode=disable'): postgres://postgres:postgres@db:5432/postgres?sslmode=disable
Audit Database schema (default 'public'):
Elasticsearch URL (default 'http://localhost:9200'):
Redis URL (default 'redis://localhost'): redis://redis
Redis URL for analytic (default 'redis://localhost/1'): redis://redis/1
config written to authgear.yaml
config written to authgear.secrets.yaml
```

`authgear.yaml` and `authgear.secrets.yaml` are generated in your working directory.

### Use PostgreSQL as search service

By default, Elasticsearch is used as the search service. Optionally, you can use PostgreSQL as the search service. Enter `postgresql` in the interactive prompt when you are asked to select a service for searching, followed by the search database configs.

```
Select a service for searching (elasticsearch, postgresql) (default 'elasticsearch'): postgresql
...
Search Database URL (default 'postgres://postgres:postgres@127.0.0.1:5432/postgres?sslmode=disable'): 
Search Database schema (default 'public'):
```

## Edit authgear.secrets.yaml

The three services run in the same network. We have to ensure Authgear can connect to PostgreSQL and Redis.

Since we do not have Elasticsearch in our docker-compose.yaml, we MUST remove the elasticsearch entry in `authgear.secrets.yaml` if it exist.

Edit `authgear.secrets.yaml` so that it looks like the following:

```yaml
secrets:
- data:
    database_schema: public
    database_url: postgres://postgres:postgres@db:5432/postgres?sslmode=disable
  key: db
- data:
    database_schema: public
    database_url: postgres://postgres:postgres@db:5432/postgres?sslmode=disable
  key: audit.db
- data:
    database_schema: public
    database_url: postgres://postgres:postgres@db:5432/postgres?sslmode=disable
  key: search.db
# Either remove or comment out this block if it exist.
# - data:
#     elasticsearch_url: http://localhost:9200
#   key: elasticsearch
- data:
    redis_url: redis://redis
  key: redis
- data:
    redis_url: redis://redis/1
  key: analytic.redis
# Other entries that are randomly generated.
# They are not listed here because they will be different.
```

## Start PostgreSQL and Redis

Authgear depends on them so they have to be started first.

```bash
docker compose build
docker compose up -d db redis
```

## Run database migration

Run the database migration:

```bash
docker compose run --rm  authgear authgear database migrate up --database-url="postgres://postgres:postgres@db:5432/postgres?sslmode=disable" --database-schema="public"
docker compose run --rm  authgear authgear audit database migrate up --database-url="postgres://postgres:postgres@db:5432/postgres?sslmode=disable" --database-schema="public"
docker compose run --rm  authgear authgear images database migrate up --database-url="postgres://postgres:postgres@db:5432/postgres?sslmode=disable" --database-schema="public"

# This is only needed if you use postgresql as the search service
docker compose run --rm  authgear authgear search database migrate up --database-url="postgres://postgres:postgres@db:5432/postgres?sslmode=disable" --database-schema="public"
```

## Get it running

Run everything with:

```bash
docker-compose up
```

## Verify everything is working

Visit [http://localhost:3000](http://localhost:3000) and try signing up as a new user!
