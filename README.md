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
- [Deployment](#deployment)
  - [How a deployment runs](#how-a-deployment-runs)
  - [Repository secrets and variables](#repository-secrets-and-variables)
  - [Prerequisites on the VM](#prerequisites-on-the-vm)
  - [Triggering a deployment](#triggering-a-deployment)

## Quickstart

### Prerequisites

- Docker Engine with the Compose plugin (`docker compose version`)

### Run

```bash
cp example.env .env          # then set SECRET_KEY and POSTGRES_PASSWORD
docker compose up --build
```

The application is available at <http://localhost:8282>. Sign up, then create an article.
The database schema is created automatically on the first start, so no manual migration
step is needed.

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

The frontend container reads two variables of its own. They are not application
settings but the address nginx forwards API requests to, and both have defaults that
match the Compose setup:

| Variable       | Required | Default   | Description |
| -------------- | -------- | --------- | ----------- |
| `BACKEND_HOST` | no       | `backend` | Host nginx proxies `/api` to. This is the Compose service name, resolved inside the Docker network. |
| `BACKEND_PORT` | no       | `8000`    | Port of that backend. |

The browser never contacts the backend directly: the Angular bundle requests the
relative path `/api`, and nginx forwards it. That keeps the bundle free of any
environment-specific address, makes CORS configuration unnecessary and leaves a single
published port.

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
| `backend` | `./backend`   | none           | Waits for the database to report healthy, applies migrations, then serves the app with gunicorn. |
| `frontend`| `./frontend`  | `8282`         | nginx serving the compiled Angular app and proxying `/api` to the backend. The only service reachable from outside. |

The backend runs as an unprivileged user (uid 10001); in the frontend image the nginx
worker processes do.

All services use `restart: on-failure:5`, so a container that exits with an error is
started again, up to five times. A container that exits cleanly stays stopped, and after
a reboot of the host the stack is started again with `docker compose up -d`.

### Customization

- **Change the port the application is served on:** edit the left-hand side of
  `"8282:80"` under `frontend.ports`. The right-hand side is the port nginx listens on
  inside the container and must stay in sync with `listen` in `nginx.conf.template`.
- **Reach the backend directly, e.g. for `curl`:** add a `ports` entry such as
  `"8000:8000"` to the `backend` service. Not needed in normal operation, since the
  frontend proxies `/api`.
- **Expose the database for a GUI client:** add a `ports` entry to the `db` service, for
  example `"5432:5432"`. Do this for local debugging only, never on a public host.
- **Point the frontend at a different backend:** set `BACKEND_HOST` and `BACKEND_PORT`
  for the `frontend` service. The values are substituted into the nginx configuration
  when the container starts.
- **Use a different PostgreSQL version:** change the tag of the `db` image. Major versions
  have incompatible data directories, so remove the volume (`docker compose down -v`)
  or migrate the data before switching.
- **Keep data across restarts:** the `postgres_data` volume survives `docker compose down`.
  Only `docker compose down -v` deletes it.
- **Run management commands:** `docker compose exec backend python manage.py <command>`,
  e.g. `createsuperuser`.

## Deployment

The images are built by a GitHub Actions workflow (`.github/workflows/deployment.yaml`)
and pushed to the GitHub Container Registry:

- `ghcr.io/raawer/conduit-backend`
- `ghcr.io/raawer/conduit-frontend`

Tags are generated by `docker/metadata-action`, so every image carries several names
that point at the same content:

| Tag | Example | Purpose |
| --- | ------- | ------- |
| commit SHA | `c11fbfab…` | Immutable. This is the tag that gets deployed, and the one to redeploy for a rollback. |
| branch | `main` | The latest build of that branch. Slashes become dashes, since tags cannot contain `/`. |
| `latest` | `latest` | Convenience, set only on builds from the default branch so that a feature branch never claims it. |

The images are built with `docker/build-push-action`, which also writes the OCI labels
produced by `metadata-action` (source repository, revision, build time) into the image.
That is why the Dockerfiles carry no `LABEL` instructions of their own. Each image uses
its own build cache scope, so backend and frontend do not overwrite each other's cache.

### How a deployment runs

The workflow has two jobs. `deploy` only starts once `build` has finished successfully,
so a broken build never reaches the server.

| Job | Runs on | What it does |
| --- | ------- | ------------ |
| `build`  | GitHub runner | Builds both images with the official Docker actions (`login-action`, `metadata-action`, `setup-buildx-action`, `build-push-action`) and pushes them to GHCR. The application is never built on the VM. |
| `deploy` | GitHub runner, over SSH | Copies `docker-compose.yaml` to the VM with `appleboy/scp-action`, then runs the deployment with `appleboy/ssh-action`: it writes `.env`, logs in to GHCR, pulls the images of this commit, starts them in detached mode and removes unused images. |

The remote script starts with `set -e`, so the first failing command ends the
deployment, the exit code travels back and the workflow is marked as failed. The
registry logout runs in its own step with `if: always()`, so no credentials are left in
`~/.docker/config.json` when a deployment fails.

### Repository secrets and variables

The runtime configuration of the deployed system lives in GitHub, not on the server.
The deployment writes it into `.env` next to the compose file on every run.

Secrets (*Settings → Secrets and variables → Actions → Secrets*):

| Secret | Content |
| ------ | ------- |
| `SSH_PRIVATE_KEY` | Private key of a key pair created for deployments only, including the `BEGIN`/`END` lines. |
| `SSH_HOST` | Address of the target VM. |
| `SSH_USER` | User the deployment connects as. |
| `SSH_FINGERPRINT` | SHA256 fingerprint of the VM's host key (`ssh-keyscan -t ed25519 <host> \| ssh-keygen -lf -`). Without it the SSH actions would accept any host key, so the workflow checks that it is set. |
| `SECRET_KEY` | Django secret key of the deployed instance. |
| `POSTGRES_PASSWORD` | Password of the database user. Changing it here does not change it in an existing database; the volume keeps the password set on first start. |

Variables (*same page, tab Variables*), readable in the logs and therefore only for
non-sensitive settings: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_HOST`,
`POSTGRES_PORT`, `DEBUG`, `DJANGO_LOG_LEVEL`, `ALLOWED_HOSTS`.

`ALLOWED_HOSTS` has to contain the address of the VM, otherwise Django answers every
request with `400`.

The deployment job checks that the two mandatory secrets are set before it touches the
server, so a missing secret fails the run instead of leaving a broken `.env` behind.

No registry credentials are stored: the workflow authenticates with the `GITHUB_TOKEN`
that GitHub issues per run, passes it to the VM over standard input and logs out again
after the deployment.

### Prerequisites on the VM

- Docker Engine with the Compose plugin
- The public key of the deployment key pair in `~/.ssh/authorized_keys`
- A directory `~/deploy/conduit`; the pipeline creates it if it does not exist

The pipeline writes both `docker-compose.yaml` and `.env` into that directory, so the
server holds no configuration of its own. Editing `.env` on the VM by hand has no
lasting effect: the next deployment overwrites it. The values are piped in over
standard input and the file is created with `umask 077`, so it is never world-readable
and the values appear neither on the runner's disk nor in the process list of the VM.

The trade-off of this approach is that the secrets of the running system are stored in
GitHub, where anyone with write access to the repository could read them through a
workflow. A production setup would keep them in a secret manager and give the pipeline
a short-lived read token instead.

### Triggering a deployment

The workflow runs on every push to `main` and can also be started by hand from the
Actions tab (`workflow_dispatch`). The images are tagged with the
commit SHA, and the deploy job passes that SHA to Compose as `IMAGE_TAG`, so the
containers on the VM run exactly the images built from that commit:

```bash
ssh <user>@<host> "cd deploy/conduit && docker compose ps"
```

The `IMAGE` column shows the deployed tag. After each deployment the VM runs `docker image prune -f`, which removes images that
lost their tag. Images still tagged with a commit SHA are kept, so a rollback does not
need to download them again.

To roll back, run Compose on the VM with the
`IMAGE_TAG` of an earlier commit:

```bash
IMAGE_TAG=<older-sha> docker compose up -d --no-build
```
