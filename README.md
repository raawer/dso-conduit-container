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

\* Required when `DEBUG` is `false`.

Generate a secret key:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

<!-- TODO: add frontend and database variables once they exist -->

### Customization

<!-- TODO: explain how to change ports, hosts and volumes in docker-compose.yaml -->
