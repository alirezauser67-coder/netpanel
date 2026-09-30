# NetPanel

Self-hosted web control panel for managing Xray-core server inbounds.

Go backend + React 19 / Ant Design 6 frontend, shipped as a single binary with
the UI embedded. Manages inbounds and clients, per-client traffic quotas and
expiry, live connection stats, share links / QR codes, a subscription server,
a Telegram bot, multi-node federation, and a REST API with in-panel docs.

> Personal use only. Make sure your usage complies with the law where you live
> and with your hosting provider's terms.

## Stack

- **Backend** — Go 1.27, Gin, GORM. Manages Xray-core as a child process.
- **Frontend** — React 19, Ant Design 6, Vite 8, TypeScript.
- **Storage** — SQLite by default, PostgreSQL optional.

## Run it

### Docker

```bash
docker compose up -d
```

The panel listens on `2053`. Data lives in `/etc/netpanel` inside the
container; mount a volume there to persist it (see `docker-compose.yml`).

### Railway

`railway.json` is included — Railway detects the Dockerfile and builds it with
no extra configuration. Set a volume on `/etc/netpanel` if you want the
database to survive redeploys.

### From source

```bash
cd frontend && npm ci && npm run build && cd ..
go build -o netpanel main.go
./netpanel
```

Requires cgo (the SQLite driver needs a C compiler).

## Configuration

All settings are environment variables; none are required.

| Variable | Description | Default |
| --- | --- | --- |
| `XUI_PORT` | Panel port | `2053` |
| `XUI_DB_TYPE` | `sqlite` or `postgres` | `sqlite` |
| `XUI_DB_DSN` | PostgreSQL connection string | — |
| `XUI_DB_FOLDER` | SQLite database directory | `/etc/netpanel` |
| `XUI_LOG_LEVEL` | `debug`, `info`, `warning`, `error` | `info` |
| `XUI_DEBUG` | Debug mode | `false` |
| `XUI_ENABLE_FAIL2BAN` | Enforce per-client IP limits with fail2ban | `true` |
| `XUI_INIT_WEB_BASE_PATH` | Initial web base path | `/` |
| `XUI_TUNNEL_HEALTH_MONITOR` | Probe the tunnel and restart Xray on repeated failure | `false` |

fail2ban bans with `iptables`, so the container needs `NET_ADMIN` and
`NET_RAW` (`cap_add` in `docker-compose.yml`).

## Layout

```
main.go               entry point + CLI (run, migrate, setting, cert)
internal/config/      env parsing, version, paths
internal/database/    GORM schema and migrations
internal/web/         Gin server, controllers, services, cron jobs
internal/xray/        Xray child process, config generation, gRPC stats
internal/sub/         subscription server (raw / JSON / Clash)
frontend/             React + TS source, built into internal/web/dist
docs/                 documentation site
```

## Credits

This project is a fork of [3x-ui](https://github.com/MHSanaei/3x-ui) by
MHSanaei, and builds on [Xray-core](https://github.com/XTLS/Xray-core).
Routing data comes from the [Iran v2ray rules](https://github.com/chocolate4u/Iran-v2ray-rules)
and [Russia v2ray rules](https://github.com/runetfreedom/russia-v2ray-rules-dat)
projects.

## License

GPL-3.0 — see [LICENSE](LICENSE).
