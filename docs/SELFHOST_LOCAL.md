# Build & run this fork (self-built local image)

The official `neuman1812/nestarr` image is rebuilt from the upstream
`tokendad/Nestarr` repository, so it does **not** include changes made on a fork.
If you customised this fork (for example, UI tweaks), build the image from the
fork's own source instead.

## Prerequisites

- Docker with `docker compose` (v2) on the target host.
- A clone of **this fork** (`trucnguyenmax/Nestarr`) on that host.

## 1. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and at minimum set:

- `SECRET_KEY` and `JWT_SECRET_KEY` — generate with
  `python -c "import secrets; print(secrets.token_urlsafe(32))"`
- `TZ` — e.g. `Asia/Ho_Chi_Minh`
- `GEMINI_API_KEY` — optional, enables the built-in AI photo detection /
  Category Agent (replaces the deprecated plugin API)
- `DISABLE_SIGNUPS=true` for family/private use

`.env` and `data/` are gitignored and will not be committed.

## 2. Build and start

```bash
docker compose -f docker-compose.local.yml up -d --build
```

The app is then served at `http://<host>:8181` (override with `APP_PORT` in
`.env`).

## 3. Verify

```bash
docker compose -f docker-compose.local.yml ps
curl -fsS http://localhost:8181/api/health
```

Data (SQLite database and media) is persisted under `./data/` in this
directory, so keep it backed up.

## 4. Update to the latest fork commit

```bash
git pull
docker compose -f docker-compose.local.yml up -d --build
```

### Updating a private fork from upstream

A private fork loses GitHub's "Sync fork" button. This fork's clone keeps two
remotes for that reason (`origin` points at `tokendad/Nestarr`, `fork` at your
private repo). To pull the latest upstream changes:

```bash
git fetch origin
git merge origin/main
git push fork main
docker compose -f docker-compose.local.yml up -d --build
```

Resolve any merge conflicts, then push again.

## Notes

- The frontend is built inside the image (`Dockerfile`), so UI changes from
  this fork are baked in at build time.
- Because the customised image only lives on your host, track what you changed
  here (git log) so you can re-apply on a fresh machine.
- Hardware features (Bluetooth asset tracking, CUPS printer integration) need
  `network_mode: host`, `privileged: true`, and extra device mounts — see the
  commented block in `docker-compose.local.yml`. Most home-inventory setups do
  not need them.