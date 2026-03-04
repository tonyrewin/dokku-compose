## Dokku Compose Builder & Scheduler Plugin

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Dokku](https://img.shields.io/badge/dokku-plugin-blue)](https://dokku.com/docs/community/plugins/)

**Dokku plugin that lets you build and run apps using Docker Compose v2** as a Dokku builder (`BUILDER_TYPE=compose`) and scheduler (`DOKKU_SCHEDULER=compose`).

### Features

- **Builder integration**: uses `docker compose build` to build all services with a `build` section.
- **Primary image selection**: chooses `web` service as primary or any service labeled `com.dokku.primary-image=true`.
- **Scheduler integration**: deploys applications using `docker compose up -d` while still triggering Dokku's post-deploy hooks.
- **Configurable compose path**: supports both relative and absolute `compose-path` properties for builder and scheduler.

### Requirements

- **Dokku** (recent version with builder/scheduler triggers).
- **Docker** with **Docker Compose v2** (`docker compose` subcommand must be available).
- **jq** installed on the Dokku host (used to validate and inspect the compose file).

### Installation

From your Dokku host, install the plugin directly from this repository:

```bash
sudo dokku plugin:install https://github.com/<your-org>/dokku-compose.git dokku-compose
```

Replace `https://github.com/<your-org>/dokku-compose.git` with the actual repository URL.

You can verify installation via:

```bash
dokku plugin:list
```

You should see an entry for `dokku-compose` (or the name you chose during install).

### Usage

#### 1. Use compose as a builder

Set the builder for an app to `compose`:

```bash
dokku builder:set <app> selected compose
```

Optionally, configure a custom compose file path (relative to the repo root or absolute on disk):

```bash
dokku builder:property-set <app> builder-compose compose-path docker-compose.prod.yml
```

Deploy as usual (`git push dokku main`), and the plugin will:

- Parse and validate the compose file.
- Build all services that have a `build` section and are not labeled `com.dokku.compose=false`.
- Tag the primary service image as `dokku/<app>:latest`.

#### 2. Use compose as a scheduler

Set the scheduler for an app to `compose`:

```bash
dokku scheduler:set <app> selected compose
```

Optionally set a custom compose path and/or project name:

```bash
dokku scheduler:property-set <app> scheduler-compose compose-path docker-compose.deploy.yml
dokku scheduler:property-set <app> scheduler-compose project-name my-dokku-project
```

On deploy, the plugin will:

- Resolve the compose file under `$DOKKU_ROOT/<app>` (or use an absolute path).
- Run `docker compose -p <project-name> up -d`.
- Trigger Dokku's `core-post-deploy` and `post-deploy` hooks.

### Compose file conventions

- **Primary service**:
  - Prefer a service named `web`, or
  - Any service with label `com.dokku.primary-image=true`.
- **Opt-out of build** for a service:

  ```yaml
  services:
    worker:
      build: .
      labels:
        com.dokku.compose: "false"
  ```
- **External services (reuse existing instances)**:
  - To **reuse an already-running instance** (for example, an existing Weaviate/Ollama or a Dokku datastore), mark the service as external:

  ```yaml
  services:
    weaviate:
      image: semitechnologies/weaviate:latest
      labels:
        com.dokku.external: "true"
  ```

  - Behavior:
    - on deploy, the plugin checks if there is already a running container with the service name (for example `weaviate`, `ollama`);
    - if such container **exists**, it is **not recreated**, only **attached to the compose project network** (`<project>_default`);
    - if there is **no container**, the service is included in the list passed to `docker compose up -d` and is created as a normal compose service in the same network.

- **External Dokku datastore instances (postgres / redis)**:
  - You can explicitly tell the plugin that an external service corresponds to a Dokku datastore instance via the `com.dokku.datastore` label:

  ```yaml
  services:
    db:
      image: postgres:15
      labels:
        com.dokku.external: "true"
        com.dokku.datastore: "postgres:mydb"  # dokku postgres:create mydb

    cache:
      image: redis:7
      labels:
        com.dokku.external: "true"
        com.dokku.datastore: "redis:mycache"  # dokku redis:create mycache
  ```

  - Logic:
    - `postgres:mydb` → expects a container `dokku.postgres.mydb` (created by the `dokku-postgres` plugin);
    - `redis:mycache` → expects a container `dokku.redis.mycache` (created by the `dokku-redis` plugin);
    - if such a container is already running, it is **not recreated**, only attached to the `<project>_default` network;
    - if no such container is found, the plugin treats the service as a normal external one and lets `docker compose up -d` create it by the compose service name, so the deploy does not break.

- **Volumes**:
  - Host bind-mounts must use **absolute** paths.
  - Relative host paths in bind mounts are rejected with a clear error message.

### Uninstall

```bash
sudo dokku plugin:uninstall dokku-compose
```

### License

This project is licensed under the **MIT License** – see [`LICENSE`](LICENSE) for details.


