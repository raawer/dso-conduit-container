# Submission Checklist

Working copy of `Conduit Deployment Checkliste.pdf`, kept up to date while the project
is built. The PDF is the version handed in; this file exists so progress is visible in
the git history. It supersedes the checklist of the Conduit Container project, which is
preserved at the tag `conduit-container-v1.0.0`.

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
- [x] Container port exposed (backend: 8000, frontend: 80)
- [x] Multi-stage build to keep the image small

### .dockerignore

- [x] One `.dockerignore` per build context listing what must not end up in the image

### docker-compose.yaml

- [x] Services defined and configured: frontend, backend (plus the database the
      application needs)
- [x] Environment configuration for both services (non-critical variables only)
- [x] Port mappings so the containers are reachable (only the frontend publishes a
      port, 8282)
- [x] Volume configuration so data survives container restarts

### .github/workflows

- [x] A `.github` directory containing a `workflows` folder for all automation workflows
- [x] A `deployment.yaml` performing the deployment to the cloud VM over SSH:
  - [x] The deployment step opens an SSH connection to the server
  - [x] The application is no longer built on the VM (images are built on the runner
        and pushed to GHCR)
  - [x] The containerized application is started with `docker compose` in detached mode
  - [x] The workflow fails when something goes wrong (commands chained with `&&`,
        the remote exit code ends the step)

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
- [x] No passwords, tokens or usernames in the code (environment variables and GitHub
      secrets instead)
- [x] No IP addresses or other sensitive information in the repository

### Code conventions

- [x] `UPPER_CASE_WITH_UNDERSCORE` for build args, environment and shell variables
- [x] `${SOME_VAR}` notation used for variable references
- [x] Default values where they make sense
- [x] Critical configuration passed in via `.env`, never committed

### Testing

- [x] Frontend reachable on the cloud VM on port 8282 after a pipeline deployment
- [x] Services restart automatically after a failure (`restart: on-failure:5`)
- [x] The application can be navigated and loads data everywhere (sign up, articles,
      comments, profile), verified after a pipeline deployment
- [x] Logs can be read via CLI and written to a file
