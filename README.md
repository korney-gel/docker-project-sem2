# Dumpling‑Store (Backend + Frontend)

> **Stack:** Go 1.21 · Vue 3 (Vue‑CLI) · Nginx · Docker / Compose · GitHub Actions

A small demo shop that serves a REST API with different types of dumplings and a single‑page frontend. The repository is fully containerised, has separate dev/prod profiles, health‑checks, resource limits and security hardening.

---

## 1  Repository Layout

```text
.
├── backend/          Go sources (Hexagonal layout)
│   └── Dockerfile    Build & runtime stages for Go binary
├── frontend/         Vue‑CLI sources and nginx.conf
│   └── Dockerfile    build · dev · runtime stages
├── docker-compose.yml
└── .github/workflows/docker.yml  CI/CD pipeline
```

---

## 2  Prerequisites

* Docker >= 20.10
* Docker Compose v2 (bundled with Docker Desktop)
* GNU make **(optional)** – for handy aliases

---

## 3  Quick Start (Production)

```bash
# build and start containers
$ docker compose up -d --build

# 🟢  http://localhost           → Vue SPA
# 🟢  http://localhost/api/products  → JSON from Go backend
```

### 3.1  Live‑reload mode

```bash
$ docker compose --profile dev up -d
# frontend-dev on http://localhost:3000 (Vite hot reload)
```

### 3.2  Scale out the frontend

```bash
$ docker compose up -d --scale frontend=3
```

Front containers share the same stateless static files, so no extra config is needed.

---

## 4  Environment Variables

| Service      | Variable          | Default | Description                 |
| ------------ | ----------------- | ------- | --------------------------- |
| **backend**  | `PORT`            |  8081   | HTTP listen port            |
|              | `APP_ENV`         |  prod   | log level, etc.             |
| **frontend** | `VUE_APP_API_URL` |  `/api` | Base URL that Nginx proxies |

Change values in `docker-compose.yml` or inject with `-e` flags.

---

## 5  Docker Images

| Component | Base image                    | Final size |
| --------- | ----------------------------- | ---------- |
| backend   | `alpine`                      | ≈ 17 MB    |
| frontend  | `nginxinc/nginx-unprivileged` | ≈ 21 MB    |

Optimisations:

* **multi‑stage builds** – builder layers removed
* Go built with `-ldflags "-s -w"`
* `npm ci && npm cache clean --force`
* `NODE_OPTIONS=--openssl-legacy-provider` only during build/dev
* `.dockerignore` to exclude docs, tests, git‑meta, etc.

---

## 6  Security Hardening

* Non‑root users (`USER 1001` Go, `USER 101` Nginx).
* `no-new-privileges`, `cap_drop: [ALL]` and `read_only: true`.
* Only required ports (8081 internal, 80 external) are exposed.
* Minimal base images (Alpine & nginx‑unprivileged).
* **Trivy** scans in CI (`CRITICAL/HIGH` fail the pipeline).
* Example Docker Secret (`db_password`) in compose file.

---

## 7  CI/CD Pipeline (GitHub Actions)

| Stage                          | Tool                               |
| ------------------------------ | ---------------------------------- |
| Checkout + QEMU/BinFmt setup   | `actions/checkout`, `setup-buildx` |
| **Build images (local, load)** | `docker/build-push-action`         |
| Scan with Trivy                | `aquasecurity/trivy-action`        |
| Push to Docker Hub             | `build-push-action` (push)         |
| Integration test via Compose   | `hoverkraft-tech/compose-action`   |

Secrets required in repo:

* `DOCKER_USER` – Docker Hub login
* `DOCKER_PASSWORD` – Docker Hub token / password

---

## 8  Troubleshooting

| Symptom                              | Fix                                                                                          |
| ------------------------------------ | -------------------------------------------------------------------------------------------- |
| `frontend` container `unhealthy`     | Check that runtime stage chosen (`target: runtime`), nginx.conf alias path, SPA files exist. |
| Browser shows `Unexpected token '<'` | JS bundle missing → verify `alias /usr/share/nginx/html/; try_files … index.html;`           |
| `/api` 502 / CORS error              | Backend not healthy yet or wrong `VUE_APP_API_URL`; depends\_on + healthchecks fix timing.   |
| CI step `go.sum not found`           | Ensure build context matches `backend/` folder.                                              |

---

## 9  Makefile shortcuts (optional)

```Makefile
build:            ## Build images locally
	docker compose build
up:               ## Start stack (prod)
	docker compose up -d
logs:             ## Tail logs
	docker compose logs -f --tail=50
clean:            ## Remove containers & volumes
	docker compose down -v
```

---

### Licence

MIT. Use, tweak and enjoy your dumplings! 🍜
