# brouter_profiles

## Working Podman Build Command (with custom lookups.dat)

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
  -v ~/maps/brouter/custom/lookups.dat:/brouter/lookups.dat:ro \
  brouter-builder
```

## Quick Explanation

### Key flags explained

- **`--userns=keep-id` + `--user "$(id -u):$(id -g)"`**  
  Runs the container as your real user and keeps UID/GID mapping. Prevents permission issues on mounted volumes.

- **`--network=none`**  
  Maximum isolation (no network needed for offline builds).

- **`--cpus=8` `--memory=20g`**  
  Hard resource limits for the heavy Java process.

### Volumes

Only required directories mounted (planet, segments, tmp, read-only SRTM + custom lookups.dat).

## Extracting & Customizing `lookups.dat`

### 1. Extract the original file from the image

```bash
podman run --rm --network=none \
  --env PLANET=germany-latest.osm.pbf \
  --volume ~/maps/brouter/planet:/planet \
  --entrypoint /bin/cat \
  brouter-builder \
  /brouter/lookups.dat > ~/maps/brouter/custom/lookups.dat
```

### 2. Add custom tags (example: `tourism=viewpoint`)

```bash
if ! grep -q '^tourism;' ~/maps/brouter/custom/lookups.dat; then
  echo 'tourism;0000000000 viewpoint' >> ~/maps/brouter/custom/lookups.dat
fi
```

### 3. Build with your custom lookups

Use the main command at the top of this file — it mounts your modified `lookups.dat`.

## What worked

- `--network=none` solved the pasta/tun error
- Volume mount of custom `lookups.dat` successfully includes new tags (e.g. `tourism=viewpoint`)

### 3. SRTM HGT to BEF

podman run --rm \
  --name brouter-convert \
  -v ~/maps/brouter/srtm1_hgt:/hgt:ro \
  -v ~/maps/brouter/srtm1_bef:/bef \
  --entrypoint java \
  brouter-builder \
  -cp /brouter/brouter.jar \
  btools.mapcreator.ElevationRasterTileConverter \
  all /hgt /bef 1