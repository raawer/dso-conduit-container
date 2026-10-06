# Submission Checklist

Working copy of `Conduit Container Checkliste.pdf`, kept up to date while the project
is built. The PDF is the version handed in; this file exists so progress is visible in
the git history.

## 1. Repository

### Files

- [x] `.gitignore` excludes everything irrelevant from the repository
- [ ] A Dockerfile for both backend and frontend (backend done, frontend missing)
- [ ] `docker-compose.yaml`
- [ ] `README.md` according to the criteria below

### Dockerfiles

- [ ] Base image fits the technology stack (backend: `python:3.5-slim`; frontend missing)
- [ ] Required environment variables configured inside the Dockerfiles
- [ ] Container port exposed (backend: 8000; frontend missing)
- [ ] Multi-stage build to keep the image small (backend: 214 MB instead of ~900 MB)

### .dockerignore

- [ ] One `.dockerignore` per build context listing what must not end up in the image
      (backend done: db file, caches, git and docker files; frontend still incomplete)

### docker-compose.yaml

- [ ] Services defined and configured: frontend, backend, database (Postgres)
- [ ] Environment configuration for both services (non-critical variables only)
- [ ] Port mappings so the containers are reachable
- [ ] Volume configuration so data survives container restarts

### README.md

- [x] Table of contents
- [x] Description of the repository: contents and purpose
- [ ] "Quickstart" section with prerequisites and short instructions
- [ ] "Usage" section covering configuration and how to modify it

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
- [ ] `${SOME_VAR}` notation used for variable references
- [x] Default values where they make sense (`DEBUG=False`, `DJANGO_LOG_LEVEL=INFO`;
      no default for `SECRET_KEY` on purpose, so the app fails fast)
- [x] Critical configuration passed in via `.env`, never committed

### Testing

- [ ] Frontend reachable on the cloud VM on port 8282
- [x] Entrypoint starts a WSGI application, not a dev server (`gunicorn conduit.wsgi:application`)
- [ ] Services restart automatically after a failure
- [ ] The application can be navigated and loads data everywhere
- [ ] Logs can be read via CLI and written to a file
      (`docker logs <container> > logs.txt`; Django logs to stdout, unbuffered)
