# GHCR Deployment Flow

This project builds and pushes a Docker image to GitHub Container Registry (GHCR) on each push to `main`.

## 1. What Happens Automatically

When code is pushed to `main`:

1. GitHub Actions builds image from `src/Dockerfile`
2. Image is pushed to `ghcr.io/<owner>/motoguide-routing`
3. Tags are published:
- `latest`
- `sha-<git-sha>`

Workflow file:
- `.github/workflows/build-and-push-ghcr.yml`

## 2. One-Time Server Setup

Run on VM:

```bash
docker login ghcr.io -u YOUR_GITHUB_USERNAME
```

When prompted for password, use a GitHub Personal Access Token with:

- `read:packages`

## 3. Pull and Run Latest Image

```bash
docker pull ghcr.io/YOUR_GITHUB_OWNER/motoguide-routing:latest

docker rm -f motoguide-routing || true

docker run -d \
  --name motoguide-routing \
  -p 127.0.0.1:8989:8989 \
  -v /home/ubuntu/motoguide-routing/src/data:/src/data \
  ghcr.io/YOUR_GITHUB_OWNER/motoguide-routing:latest
```

## 4. Safer Production Deploy (Pinned SHA Tag)

Prefer a commit tag over `latest`.

```bash
docker pull ghcr.io/YOUR_GITHUB_OWNER/motoguide-routing:sha-<commit_sha>

docker rm -f motoguide-routing || true

docker run -d \
  --name motoguide-routing \
  -p 127.0.0.1:8989:8989 \
  -v /home/ubuntu/motoguide-routing/src/data:/src/data \
  ghcr.io/YOUR_GITHUB_OWNER/motoguide-routing:sha-<commit_sha>
```

## 5. Verify Runtime

```bash
docker ps --filter name=motoguide-routing
docker logs --tail=200 motoguide-routing
```

Expected startup signal:

- `loaded graph at:data/graph-cache`

If you see this instead, cache is not being reused:

- `start creating graph from data/austria-260116.osm.pbf`

## 6. Keep This Stable

To avoid cache mismatch and server-side heavy recalculation, keep these aligned between build source and server runtime:

1. GraphHopper version in `src/Dockerfile`
2. Routing settings in `src/config.yml`
3. Custom models in `src/custom-models`
4. Mounted host cache in `/home/ubuntu/motoguide-routing/src/data`
