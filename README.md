# Novu on the CloudShift Openship homelab

Self-hosted [Novu](https://github.com/novuhq/novu) v3.19.0 (notification infrastructure: in-app inbox, email, SMS, push, chat), deployed by Openship to the homelab server from `docker-compose.yml`.

| Service | Public URL | Container port |
|---|---|---|
| Dashboard | https://novu.app.cloudshiftsolutions.in | 3042 |
| API | https://novu-api.app.cloudshiftsolutions.in | 3040 |
| WebSocket (Inbox) | https://novu-ws.app.cloudshiftsolutions.in | 3041 |
| Worker, MongoDB, Redis | internal only | 3004 / 27017 / 6379 |

Traffic: browser → Vercel DNS wildcard → Hetzner VPS Caddy (TLS) → Tailscale → Openship edge → container.

## Memory budget

About 2 GB idle, 4 GB worst case. Limits: MongoDB 1.5 GB (WiredTiger cache 0.5 GB), API 1 GB, worker 1 GB, ws 512 MB, dashboard 256 MB, Redis 256 MB.

## Secrets

Set as literal values on each Openship service (service values override project env), never committed:

- `JWT_SECRET` (api, ws)
- `NOVU_SECRET_KEY` (api)
- `STORE_ENCRYPTION_KEY`, exactly 32 characters (api, worker, ws)
- `MONGO_INITDB_ROOT_PASSWORD` (mongodb) and `MONGO_URL` (api, worker, ws):
  `mongodb://novu:<password>@mongodb:27017/novu-db?authSource=admin`

`DISABLE_USER_REGISTRATION=true` is set on the api service once the admin account exists.

## Using it from an app

```ts
import { Novu } from '@novu/api';

const novu = new Novu({
  secretKey: process.env.NOVU_SECRET_KEY,
  serverURL: 'https://novu-api.app.cloudshiftsolutions.in',
});
```

Inbox component: `backendUrl="https://novu-api.app.cloudshiftsolutions.in"`, `socketUrl="https://novu-ws.app.cloudshiftsolutions.in"`.

## Upgrading

Bump the four `ghcr.io/novuhq/novu/*` tags together, push, and redeploy from Openship.
