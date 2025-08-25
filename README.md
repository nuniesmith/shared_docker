# shared-docker

Service-agnostic shared component extracted from monolith paths:

- `shared/docker`

## Added: Central Compose Stack

The unified compose definitions from `fks/compose` were migrated here under `compose/` to enable reuse across repositories and CI workflows without coupling to the monolith root path assumptions.

Usage example from repository root:

```bash
docker compose -f shared/shared-docker/compose/docker-compose.yml up -d
```

Dev overrides:

```bash
docker compose -f shared/shared-docker/compose/docker-compose.yml \
			   -f shared/shared-docker/compose/docker-compose.dev.yml up -d
```

Prod overrides:

```bash
docker compose -f shared/shared-docker/compose/docker-compose.yml \
			   -f shared/shared-docker/compose/docker-compose.prod.yml up -d
```

Ensure relative build contexts still resolve (they reference `../../fks/...`). If this repository is used standalone, replace those paths or publish built images and switch to `image:` references only.


Source snapshot commit: `UNKNOWN`

Generated: 2025-08-22T10:02:30.290009Z

Automation: scripts/extract_neutral_shared.py
