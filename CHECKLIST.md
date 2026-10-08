# Submission Checklist

Working copy of `Conduit Container Checkliste.pdf`, kept up to date while the project
is built. The PDF is the version handed in; this file exists so progress is visible in
the git history.

## 1. Repository

### Files

- [x] `.gitignore` excludes everything irrelevant from the repository
- [x] A Dockerfile for both backend and frontend
- [x] `docker-compose.yaml`
- [x] `README.md` according to the criteria below

### Dockerfiles

- [x] Base image fits the technology stack (backend: `python:3.5-slim`,
      frontend: `node:20-alpine` for the build and `nginx:alpine` to serve)
- [x] Required environment variables configured inside the Dockerfiles
      (backend: `PYTHONUNBUFFERED`, `PYTHONDONTWRITEBYTECODE`;
      frontend: `BACKEND_HOST`, `BACKEND_PORT`, `NGINX_ENVSUBST_FILTER`)
- [x] Container port exposed (backend: 8000, frontend: 80)
- [x] Multi-stage build to keep the image small (backend: 51 MB; frontend: 94 MB,
      with 327 MB of `node_modules` left behind in the build stage)

### .dockerignore

- [x] One `.dockerignore` per build context listing what must not end up in the image
      (backend: db file, caches, git and docker files; frontend: `node_modules`,
      `dist`, `.angular`, git, IDE and documentation files)

### docker-compose.yaml

- [x] Services defined and configured: frontend, backend, database (Postgres)
- [x] Environment configuration for both services (non-critical variables only)
- [x] Port mappings so the containers are reachable (only the frontend publishes a
      port, 8282; backend and database stay inside the Compose network)
- [x] Volume configuration so data survives container restarts (named volume
      `postgres_data`)

### README.md

- [x] Table of contents
- [x] Description of the repository: contents and purpose
- [x] "Quickstart" section with prerequisites and short instructions
- [x] "Usage" section covering configuration and how to modify it

## 2. Documentation

- [x] Documentation lives in the repository as README files
- [x] Documentation language is English

## 3. Notes

### General

- [ ] Loom video (max. 5 min) recorded and provided

### Security

- [x] No SSH keys in the repository
- [x] No passwords, tokens or usernames in the code (environment variables instead)
- [x] No IP addresses or other sensitive information in the repository

### Code conventions

- [x] `UPPER_CASE_WITH_UNDERSCORE` for build args, environment and shell variables
- [x] `${SOME_VAR}` notation used for variable references
- [x] Default values where they make sense (`DEBUG=False`, `DJANGO_LOG_LEVEL=INFO`;
      no default for `SECRET_KEY` on purpose, so the app fails fast)
- [x] Critical configuration passed in via `.env`, never committed
      (`SECRET_KEY` and `POSTGRES_PASSWORD` are mandatory and have no default)

### Testing

- [x] Frontend reachable on the cloud VM on port 8282
- [x] Entrypoint starts a WSGI application, not a dev server (`gunicorn conduit.wsgi:application`)
- [x] Services restart automatically after a failure (`restart: always`)
- [x] The application can be navigated and loads data everywhere (verified locally:
      sign up, create article, comments, profile)
- [x] Logs can be read via CLI and written to a file
      (`docker logs <container> > logs.txt`; Django logs to stdout, unbuffered)
