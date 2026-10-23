# Immich (personal fork)

Personal, self-hosted build of [Immich](https://github.com/immich-app/immich) used as a photo archive.
Upstream documentation: <https://docs.immich.app>.

## Images

- `ghcr.io/arsgab/immich-server`
- `ghcr.io/arsgab/immich-machine-learning` (CPU only)

Both are built by `.github/workflows/docker.yml`.

## Deploy

Use `docker/docker-compose.yml` together with `docker/example.env`:

```sh
cd docker
cp example.env .env
docker compose up -d
```

## License

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](LICENSE)

Licensed under the GNU AGPL v3, same as upstream.
