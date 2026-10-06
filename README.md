# Conduit Container

This repository contains the Conduit application, an Angular frontend and a Django REST backend, together with the Docker configuration to run it as a multi-container setup with Docker Compose and PostgreSQL. Its purpose is to build, configure and deploy the full-stack application in containers, with all sensitive settings supplied through environment variables.

## Table of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Run](#run)
- [Usage](#usage)
  - [Configuration](#configuration)
  - [Customization](#customization)

## Quickstart

### Prerequisites

<!-- TODO: list required tools and versions (Docker, Docker Compose) -->

### Run

<!-- TODO: minimal steps once docker-compose.yaml exists: clone, copy example.env to .env, docker compose up -->

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

Generate a secret key:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

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

### Customization

<!-- TODO: explain how to change ports, hosts and volumes in docker-compose.yaml -->
