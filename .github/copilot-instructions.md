# Copilot Instructions for ioq3-server

## Purpose
Keep guidance for contributors and automation about how this repository is built, run, and debugged. Update this file when Docker build, startup behavior, or service names change.

## Project Overview
- Builds and runs a dedicated ioquake3 server inside Docker using a multi-stage `Dockerfile` and a `docker-compose.yml` stack.
- Supports multiple server modes (FFA, CTF, Missionpack) via separate compose services.
- Configs and runtime behavior are injected at container startup by `files/entrypoint.sh`.

## Quick Ops
- Build image(s): `docker compose build` or `docker build -t ioq3-server .`
- Run stack: `docker compose up`
- To run a single service for debugging: `docker compose up landing` or `docker compose up fastdl`

## Startup & Runtime Behavior
- Entrypoint: `files/entrypoint.sh` copies configs into the game dir, sets env vars, and launches the dedicated server binary.
- Configs: Place editable server configs in `configs/` (and subfolders like `configs/osp/`). At container start these are copied into the runtime `baseq3/` path.
- Environment variables:
  - `SERVER_ARGS` — example: `+exec server-ffa.cfg` to choose the config to exec.
  - `SERVER_MOTD` — message of the day injected at runtime.
  - `ADMIN_PASSWORD` — set to a value or let entrypoint generate a random one.
  - `FASTDL_URL` — when set, entrypoint will configure `sv_allowDownload 1` and `sv_dlURL` to this URL.
- Data mounts:
  - Game data directories (`baseq3/`, `missionpack/`) are expected to contain `.pk3` files but are kept out of git; these are typically mounted read-only into containers.
  - Configs are mounted read-write so users can edit and persist their server configuration.

## FastDL and Landing
- `fastdl/` contains an nginx config (`fastdl/nginx.conf`) and `fastdl/public/` where you place custom downloadable `.pk3` files. Do NOT place the official `pak0.pk3` in `fastdl/public/` — serve that from a proper FastDL host if needed.
- `landing/` is a small Node/Express status page (uses `gamedig`) that queries servers over UDP and is served on port 8081 by default in the compose stack. Configure which servers are shown via the `SERVERS_JSON` env in `docker-compose.yml`.

## Config Management & Required Game Data
- Add official game `pak0.pk3` to `baseq3/` and `missionpack/` on hosts or mounts before starting the server — the repo does not include these files.
- Default config templates live in `configs/default-configs/` (e.g., `server-ffa.cfg`, `server-ctf.cfg`). Copy or extend them in `configs/` for your servers.

## Security & Runtime Notes
- Containers run as a non-root user (`ioq3ded`) for safety.
- Final images only contain the dedicated server binary and required runtime files (minimal attack surface).

## Debugging & Logs
- Server logs and entrypoint diagnostics are printed to stdout/stderr (view with `docker compose logs -f <service>`).
- The entrypoint script prints helpful diagnostics if the dedicated server binary is missing or ambiguous; check permissions and mounted paths when the server fails to start.

## Key Files & Directories (at-a-glance)
- `Dockerfile` — multi-stage build for the dedicated server binary.
- `docker-compose.yml` — composes multiple server instances, `fastdl`, and `landing` services.
- `files/entrypoint.sh` — startup, config copy, env var handling, admin password generation.
- `configs/` — writable configs for servers (e.g., `server-ffa.cfg`).
- `configs/default-configs/` — templates provided for common server types.
- `baseq3/`, `missionpack/` — game asset folders (user-supplied `.pk3` files).
- `fastdl/nginx.conf` and `fastdl/public/` — FastDL serving config and files.
- `landing/` — status page (Node/Express) and its Dockerfile.

## External Dependencies
- Base image uses Alpine tooling; building the server requires typical build tools (git, curl, gcc, cmake, ninja).

## Contributor Tips
- Before opening issues or PRs: include `docker compose build` and `docker compose up` logs and the contents of any modified `configs/` files.
- If changing startup behavior, update this file to keep guidance and automation in sync.

## Checklist Before Running
- Ensure `pak0.pk3` is available in `baseq3/` and `missionpack/` (not tracked in git).
- Set `FASTDL_URL` if you want clients to download custom maps/mods from your HTTP server.
- Confirm any custom `configs/` are present and referenced by `SERVER_ARGS`.

---


**Review and update this file as project conventions evolve. If any section is unclear or missing, please provide feedback for improvement.**
