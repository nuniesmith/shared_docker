# Docker Consolidation Migration Guide

This guide describes how to migrate individual service Dockerfiles to the unified `shared/shared_docker/Dockerfile` and optional thin wrappers.

## Service Classification Matrix

| Category | Examples | Action |
|----------|----------|--------|
| Python API/Data/Engine/Worker | fks_api, fks_data, fks_engine, fks_worker, fks_execution | Use shared build directly (compose build args) |
| Python GPU | fks_training, fks_transformer | Use shared build with `BUILD_TYPE=gpu` |
| Node/Web | fks_web, personal web clients | Use shared build with `SERVICE_RUNTIME=node` |
| Rust Services | fks_config, fks_node_network | Use shared build with `SERVICE_RUNTIME=rust` (enhance shared Dockerfile if needed) |
| .NET / Mono | fks_ninja | Defer – evaluate converting to shared dotnet stage |
| Infra (nginx, postgres, redis, syncthing) | fks_nginx (wrapper), postgres, redis, sync containers | Prefer vendor image (use `image:` not build) |
| Custom game/server (heavy bespoke) | clonehero server, 2009scape, ats | Keep custom for now; consider parameterizing later |

## Migration Levels

1. Compose-only (Preferred): Remove per-service Dockerfile. In compose: set `dockerfile: ../../shared/shared_docker/Dockerfile` and appropriate `args`.
2. Thin Wrapper: Minimal Dockerfile containing only metadata or minor overrides:

```Dockerfile
ARG BASE_TAG=shared-base:latest
FROM ${BASE_TAG} AS runtime
LABEL org.fks.wrapper="true" org.fks.service="fks_api"
# (Optional) COPY overrides
```
3. Full Custom (Avoid): Retain legacy multi-stage only if requirements unsupported in shared base.

## Step-by-Step Migration (Example: fks_api)

1. Ensure compose service uses shared dockerfile path (already applied in `fks_master/docker-compose.yml`).
2. Delete or archive original `fks/fks_api/Dockerfile`:
   - Move to `fks/fks_api/Dockerfile.legacy` temporarily until confidence gained.
3. Build with helper:

```bash
shared/shared_docker/scripts/build-service.sh fks/fks_api --runtime python --type api --tag fks_api:dev
```

4. Run container and smoke test endpoint:

```bash
docker run --rm -p 8000:8000 fks_api:dev curl -sf http://localhost:8000/health || echo FAIL
```

5. If OK: remove `Dockerfile.legacy` and commit.

## Build Helper Usage

`build-service.sh` centralizes arguments:

```bash
./shared/shared_docker/scripts/build-service.sh fks/fks_training --runtime python --type training --gpu --tag fks_training:gpu
```

Add custom build arg:

```bash
./shared/shared_docker/scripts/build-service.sh fks/fks_api --runtime python --type api --arg REQUIREMENTS_FILE=requirements_prod.txt
```

## Validation Checklist

- [ ] Image builds successfully
- [ ] Healthcheck passes (or manual curl works)
- [ ] Logs directory writable
- [ ] GPU services: torch / CUDA availability confirmed
- [ ] No missing runtime deps (inspect container logs)

## Rollback

If issues arise, revert compose change and restore legacy Dockerfile from git history or `Dockerfile.legacy`.

## Future Enhancements

- Parameterize DOTNET / Rust builder toggles via compose profiles
- Add SBOM generation stage
- Add automated diff check between legacy and shared image package inventories.

---
Generated: $(date -u +%Y-%m-%d)
