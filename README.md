# container-images

Upstream container images rebuilt with a different base than upstream ships, published to GHCR.
linux/amd64 only. Tags are `<upstream version>-<variant>`.

| Image | Upstream | Variant |
|---|---|---|
| `ghcr.io/connorsapps/cephcsi` | [ceph/ceph-csi](https://github.com/ceph/ceph-csi) | `-el9`: Rocky Linux 9 base. Upstream v3.18+ images are EL10 and need x86-64-v3 (AVX2); this runs on x86-64-v2 CPUs (Sandy/Ivy Bridge). |

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
