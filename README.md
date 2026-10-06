# Conduit Container

This repository contains the Conduit application, an Angular frontend and a Django REST backend, together with the Docker configuration to run it as a multi-container setup with Docker Compose and PostgreSQL. Its purpose is to build, configure and deploy the full-stack application in containers, with all sensitive settings supplied through environment variables.

## Table of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Run](#run)
- [Usage](#usage)
  - [Configuration](#configuration)
  - [Platform](#platform)
  - [Services](#services)
  - [Customization](#customization)

## Quickstart

### Prerequisites

- Docker Engine with the Compose plugin (`docker compose version`)

### Run

```bash
cp example.env .env          # then set SECRET_KEY and POSTGRES_PASSWORD
docker compose up --build
```

The backend is available at <http://localhost:8000/api/articles>. The database schema is
created automatically on the first start, so no manual migration step is needed.

Generate a secret key:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

Stop the stack with `docker compose down`, or with `docker compose down -v` to delete
the database volume as well.

## Usage

### Configuration

All configuration is passed to the containers via environment variables. Copy `example.env` to `.env` and adjust the values. Never commit the `.env` file.

| Variable        | Required | Default | Description                                                                                   |
| --------------- | -------- | ------- | --------------------------------------------------------------------------------------------- |
| `SECRET_KEY`    | yes      | –       | Django secret key used for cryptographic signing. Generate a new random value for every setup. |
| `DEBUG`         | no       | `false` | Enables Django debug mode when set to `true`. Must be `false` on any publicly reachable host.   |
| `ALLOWED_HOSTS` | yes*     | –       | Comma-separated list of host names or IPs the backend responds to, e.g. `localhost,127.0.0.1`. |
| `DJANGO_LOG_LEVEL` | no    | `INFO`  | Log level for Django's output on stdout. Keep `INFO` in normal operation: on `DEBUG` every SQL statement is logged, including the values it carries. |
| `POSTGRES_DB`   | yes      | –       | Name of the database. Read by both the database container (which creates it on first start) and the backend. |
| `POSTGRES_USER` | yes      | –       | Database user, created on the first start of the database container. |
| `POSTGRES_PASSWORD` | yes  | –       | Password for that user. Set a value of your own; the application refuses to start without it. |
| `POSTGRES_HOST` | no       | `db`    | Host the backend connects to. Inside Compose this is the service name of the database, not `localhost`. |
| `POSTGRES_PORT` | no       | `5432`  | Port of the database server. |

\* Required when `DEBUG` is `false`.

Variables without a default are mandatory: the backend raises `ImproperlyConfigured`
on startup when one of them is missing, rather than starting in a half-configured state.

The backend logs to stdout, so container logs are read with `docker logs <container>`
and can be written to a file with `docker logs <container> > logs.txt`. Errors are
logged regardless of the `DEBUG` setting.

<!-- TODO: add frontend variables once the frontend image exists -->

### Platform

The backend images are built for `linux/amd64`:

```bash
docker build --platform linux/amd64 -t conduit-backend ./backend
```

Django 1.10 requires Python 3.5, and no prebuilt `psycopg2` wheel exists for that
Python version on `arm64`. Building for `amd64` uses the available wheel and matches
the target VM; on an Apple Silicon machine the image runs under emulation, which
affects build and start-up speed only.

Python 3.5 and Debian Buster have both reached end of life, so the base image receives
no security updates. This is accepted here because the application is pinned to
Django 1.10; a real deployment would upgrade the stack first.

### Services

| Service   | Image / build | Published port | Notes |
| --------- | ------------- | -------------- | ----- |
| `db`      | `postgres:17` | none           | Only reachable from inside the Compose network. Data lives in the named volume `postgres_data`. |
| `backend` | `./backend`   | `8000`         | Waits for the database to report healthy, applies migrations, then serves the app with gunicorn. |

Both services use `restart: always`, so a container that exits because of an error is
started again.

### Customization

- **Change the published backend port:** edit the left-hand side of `"8000:8000"` under
  `backend.ports`. The right-hand side is the port inside the container and must stay in
  sync with `EXPOSE` and the `--bind` argument in the Dockerfile.
- **Expose the database for a GUI client:** add a `ports` entry to the `db` service, for
  example `"5432:5432"`. Do this for local debugging only, never on a public host.
- **Use a different PostgreSQL version:** change the tag of the `db` image. Major versions
  have incompatible data directories, so remove the volume (`docker compose down -v`)
  or migrate the data before switching.
- **Keep data across restarts:** the `postgres_data` volume survives `docker compose down`.
  Only `docker compose down -v` deletes it.
- **Run management commands:** `docker compose exec backend python manage.py <command>`,
  e.g. `createsuperuser`.
