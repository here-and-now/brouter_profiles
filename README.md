# brouter_profiles


# BRouter Builder - Podman Command

## The Command

```bash
podman run --rm \
  --name brouter-build \
  --network=none \
  --userns=keep-id \
  --user "$(id -u):$(id -g)" \
  --cpus=8 \
  --memory=20g \
  --env PLANET=germany-latest.osm.pbf \
  --env PLANET_UPDATE=0 \
  --env JAVA_OPTS="-Xmx8192M -Xms8192M -Xmn512M" \
  -v ~/maps/brouter/tmp:/brouter-tmp \
  -v ~/maps/brouter/planet:/planet \
  -v ~/maps/brouter/srtm1:/srtm1:ro \
  -v ~/maps/brouter/segments:/segments \
  brouter-builder
```

## Quick Explanation

**Key flags explained:**

- `--userns=keep-id` + `--user "$(id -u):$(id -g)"`  
  Runs the container as **your real user** (not root) and keeps UID/GID mapping identical to the host. This prevents permission problems on the mounted volumes.

- `--network=none`  
  No network access at all — maximum isolation.

- `--cpus=8 --memory=20g`  
  Hard resource limits so the heavy Java build can't overwhelm your machine.

- Volumes  
  Only the directories you need are mounted (`planet`, `segments`, `tmp`, read-only SRTM).


