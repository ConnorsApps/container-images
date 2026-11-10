# container-images

Upstream container images rebuilt with a different base than upstream ships, published to GHCR.
Some are built from source into `scratch` (a `Dockerfile` per directory).
linux/amd64 only, baseline x86-64 (no AVX2). Tags are `<upstream version>-<variant>`.

| Image | Upstream | Variant |
|---|---|---|
| `ghcr.io/connorsapps/cephcsi` | [ceph/ceph-csi](https://github.com/ceph/ceph-csi) | `-el9`: Rocky Linux 9 base. Upstream v3.18+ images are EL10 and need x86-64-v3 (AVX2); this runs on x86-64-v2 CPUs (Sandy/Ivy Bridge). |
| `ghcr.io/connorsapps/wireguard-exporter` | [MindFlavor/prometheus_wireguard_exporter](https://github.com/MindFlavor/prometheus_wireguard_exporter) | `-static`: exporter + `wg` in `scratch` (~3 MB). |
| `ghcr.io/connorsapps/nftables-exporter` | [ConnorsApps/nftables-exporter](https://github.com/ConnorsApps/nftables-exporter) | `-static`: exporter + static `nft` in `scratch` (~13 MB). Tag is the short commit SHA. |

Router images live here, never in Harbor (it sits behind the router). Set each package public.

## cephcsi

Upstream's `deploy/cephcsi/image/Dockerfile`, unmodified, with el9 build args:

```sh
git clone --depth 1 --branch v3.18.1 https://github.com/ceph/ceph-csi.git
cd ceph-csi
podman build -f deploy/cephcsi/image/Dockerfile . \
  -t ghcr.io/connorsapps/cephcsi:v3.18.1-el9 \
  --label org.opencontainers.image.source=https://github.com/ConnorsApps/container-images \
  --build-arg GO_ARCH=amd64 \
  --build-arg BASE_IMAGE=quay.io/rockylinux/rockylinux:9 \
  --build-arg FINAL_BASE_IMAGE=quay.io/rockylinux/rockylinux:9-minimal \
  --build-arg CEPH_RELEASE_RPM=https://download.ceph.com/rpm-tentacle/el9/noarch/ceph-release-1-1.el9.noarch.rpm \
  --build-arg CSI_IMAGE_NAME=ghcr.io/connorsapps/cephcsi \
  --build-arg CSI_IMAGE_VERSION=v3.18.1 \
  --build-arg GIT_COMMIT="$(git rev-list -1 HEAD)"
```

With Rook, set `csi.cephcsi.repository` / `csi.cephcsi.tag` in the operator chart values to use it.

## wireguard-exporter

Needs host network and `CAP_NET_ADMIN`.

```sh
podman build ./wireguard-exporter \
  -t ghcr.io/connorsapps/wireguard-exporter:3.6.6-static \
  --label org.opencontainers.image.source=https://github.com/ConnorsApps/container-images \
  --build-arg EXPORTER_VERSION=3.6.6 \
  --build-arg WG_TOOLS_VERSION=1.0.20250521
```

## nftables-exporter

Needs host network and `CAP_NET_ADMIN`. Static `nft` (`NFTABLES_VERSION`, default 1.1.6) at `/sbin/nft`.
`REF` is a full SHA. Use `48a36b2` (unmerged `feat/terse-mode`): `nft -t` keeps GeoIP set elements
out of each scrape; master lacks it.

```sh
podman build ./nftables-exporter \
  -t ghcr.io/connorsapps/nftables-exporter:48a36b2-static \
  --label org.opencontainers.image.source=https://github.com/ConnorsApps/container-images \
  --build-arg REF=48a36b23db45c93210c0126533fe93bde7b0706a
```
