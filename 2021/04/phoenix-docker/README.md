# Deploy Phoenix with Docker on gigalixir

[Gigalixir - Guide](https://gigalixir.readthedocs.io/en/latest/getting-started-guide.html)

Create a new gaigalixir app

`gigalixir create -n [project_name] --cloud gcp --region europe-west1`

```

> gigalixir create [project_name]
> git remote -v

gigalixir https://git.gigalixir.com/[project_name]git/ (fetch)
gigalixir https://git.gigalixir.com/[project_name].git/ (push)
origin https://github.com/sujinleeme/[project_name].git (fetch)
origin https://github.com/sujinleeme/[project_name].git (push)

```

**Troubleshooting**

Fix node modules cp error with clean_cache ([See)](https://github.com/gjaldon/heroku-buildpack-phoenix-static/issues/80)

Add `clean_cache=true` in `phoenix_static_buildpack.config`

```

# Clean out cache contents from previous deploys

clean_cache=true

```

In `Dockerfile`:

```

FROM ubuntu-elixir as build

# ===========
# Application
# ===========

# prepare build dir

WORKDIR /app

# install hex + rebar

RUN mix local.hex --force && \
 mix local.rebar --force

# set build ENV

ENV MIX_ENV=prod

# install mix dependencies

COPY mix.exs mix.lock ./
COPY config config
RUN mix do deps.get, deps.compile

# build assets

COPY assets/package.json assets/package-lock.json ./assets/
RUN npm --prefix ./assets ci --progress=false --no-audit --loglevel=error

COPY priv priv
COPY assets assets
RUN npm run --prefix ./assets deploy
RUN mix phx.digest

# compile and build release

COPY lib lib

# uncomment COPY if rel/ exists

# COPY rel rel

RUN mix do compile, release

FROM scratch AS app

WORKDIR /app
COPY --from=build /app/\_build/prod/app_name-\*.tar.gz ./

CMD ["/bin/bash"]

```

In `docker-compose.yml` :

```

version: '3.6'
services:
db:
environment:
PGDATA: /var/lib/postgresql/data/pgdata
POSTGRES_PASSWORD: postgres
POSTGRES_USER: postgres
POSTGRES_HOST_AUTH_METHOD: trust
image: 'postgres:11-alpine'
restart: always
volumes: - 'pgdata:/var/lib/postgresql/data'
web:
build: .
depends_on: - db
environment:
MIX_ENV: dev
env_file: - .env
ports: - '4000:4000'
volumes: - .:/app
volumes:
pgdata:
```
