# ezbookkeeping for RISC-V (linux/riscv64)

Unofficial automated builds of [ezbookkeeping](https://github.com/mayswind/ezbookkeeping)
for the `linux/riscv64` architecture.

**No patches are applied to ezbookkeeping.** The upstream source is cloned at
the release tag and built directly. This repository contains only the build
infrastructure (Dockerfile + CI workflow).

---

## How it works

| Step | Detail |
|------|--------|
| **Frontend** | Node 24 builds the Vue/TypeScript UI natively on amd64 (no QEMU) |
| **Backend** | `gcc-riscv64-linux-gnu` cross-compiles the Go+CGO binary on amd64 (no QEMU) |
| **CGO / SQLite** | `go-sqlite3` embeds the SQLite amalgamation; no extra riscv64 library package needed |
| **Runtime image** | `ubuntu:24.04` (glibc, compatible with the dynamically-linked binary) |
| **Automation** | Daily cron checks for new upstream releases and skips already-built versions |

---

## Usage

### Docker

```sh
docker pull <your-dockerhub-username>/ezbookkeeping-riscv64:latest
```

```sh
docker run -d \
  --name ezbookkeeping \
  -p 8080:8080 \
  -v ./data:/ezbookkeeping/data \
  <your-dockerhub-username>/ezbookkeeping-riscv64:latest
```

Open `http://<device-ip>:8080` in a browser.

### Docker Compose

```yaml
services:
  ezbookkeeping:
    image: <your-dockerhub-username>/ezbookkeeping-riscv64:latest
    ports:
      - "8080:8080"
    volumes:
      - ./data:/ezbookkeeping/data
    restart: unless-stopped
```

### Custom configuration

Mount your own config file and point to it via the `EBK_CONF_PATH` environment
variable:

```sh
docker run -d \
  -p 8080:8080 \
  -v ./data:/ezbookkeeping/data \
  -v ./ezbookkeeping.ini:/etc/ezbookkeeping.ini:ro \
  -e EBK_CONF_PATH=/etc/ezbookkeeping.ini \
  <your-dockerhub-username>/ezbookkeeping-riscv64:latest
```

See the [ezbookkeeping configuration docs](https://ezbookkeeping.mayswind.net/configuration)
for all available options.

---

## GitHub Releases

Each release is tagged `<version>-riscv64` (e.g. `v1.4.0-riscv64`) and
includes:

- `ezbookkeeping-<version>-linux-riscv64.tar.gz` — standalone binary archive
- `SHA256SUMS` — checksums file

---

## Verifying a binary

```sh
tar -xzf ezbookkeeping-*-linux-riscv64.tar.gz
file ./ezbookkeeping
# → ELF 64-bit LSB pie executable, UCB RISC-V, ...
```

---

## Building locally

```sh
docker buildx build \
  --platform linux/riscv64 \
  -f Dockerfile.ezbookkeeping \
  --build-arg EZB_VERSION=v1.4.0 \
  -t ezbookkeeping-riscv64:v1.4.0 \
  --load \
  .
```

---

## Repository secrets and variables

Required to publish Docker images and GitHub Releases via the CI workflow:

| Type | Name | Description |
|------|------|-------------|
| Variable | `DOCKERHUB_USERNAME` | Docker Hub username |
| Secret | `DOCKERHUB_TOKEN` | Docker Hub access token |

`GITHUB_TOKEN` is provided automatically by GitHub Actions.

---

## Triggering a manual build

1. Go to **Actions → Build ezbookkeeping for RISC-V (riscv64)**
2. Click **Run workflow**
3. Optionally set a specific version (e.g. `v1.4.0`) or enable **Force rebuild**

---

## License

The build infrastructure in this repository is licensed under the
[MIT License](LICENSE).

ezbookkeeping itself is © mayswind and contributors, also
[MIT licensed](https://github.com/mayswind/ezbookkeeping/blob/master/LICENSE).
