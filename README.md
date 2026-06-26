# Sovereign Hive — Queen Repository

This is the Queen of the Sovereign Hive Federation. It holds the canonical constitution (`soul.md`), the colony manifest (`hive.yml`), and the governance workflows that keep all colonies in sync.

## What lives here

| File | Purpose |
|------|---------|
| `soul.md` | Immutable constitution — the source of truth for all colonies |
| `hive.yml` | Registry of all 9 colonies with roles, URLs, health paths |
| `.github/workflows/constitution-sync.yml` | Dispatches `constitution_update` to all live colonies when `soul.md` changes |
| `.github/workflows/hive-health.yml` | Polls all colony `/colony/health` endpoints every 30 min |
| `.gitmodules` | All colonies registered as git submodules under `colonies/` |

## Colonies

| Name | Role | Repo | Health Path |
|------|------|------|-------------|
| THEHIVE | core | TehutiRaEl/THEHIVE | `/colony/health` |
| aether | revenue | TehutiRaEl/aether | `/api/colony/health` |
| automatisch | automation | TehutiRaEl/automatisch | `/colony/health` (sidecar :3030) |
| kimi-gateway | llm-gateway | TehutiRaEl/Kimi-K2 | `/colony/health` (:3333) |
| localagi | inference | TehutiRaEl/LocalAGI | `/colony/health` |
| academy-books | knowledge | TehutiRaEl/free-programming-books | static |
| academy-camp | curriculum | TehutiRaEl/freeCodeCamp | static |
| outer-colony-a | outer-colony | TehutiRaEl/NAR2 | `/colony/health` |
| outer-colony-b | outer-colony | TehutiRaEl/4DBRAIN | `/colony/health` |

## Setup

### 1. Set the `COLONY_DISPATCH_TOKEN` secret

The `constitution-sync` workflow uses `repository_dispatch` to push constitution updates to all live colonies. It needs a PAT with `repo` scope.

1. Create a GitHub Personal Access Token (classic) with `repo` scope at https://github.com/settings/tokens
2. Go to **Settings → Secrets and variables → Actions** in this repo
3. Add a new repository secret named `COLONY_DISPATCH_TOKEN` with the PAT value

### 2. Set colony URL secrets (for health monitoring)

For `hive-health.yml` to poll live colonies, add these secrets (set to base URL, no trailing slash):

| Secret | Example |
|--------|---------|
| `THEHIVE_URL` | `https://thehive.yourdomain.com` |
| `AETHER_URL` | `https://aether.vercel.app` |
| `AUTOMATISCH_COLONY_URL` | `http://automatisch.yourdomain.com:3030` |
| `KIMI_COLONY_URL` | `http://kimi.yourdomain.com:3333` |
| `LOCALAGI_URL` | `http://localagi.yourdomain.com` |
| `NAR2_URL` | `http://nar2.yourdomain.com` |
| `FOURBRAIN_URL` | `http://4dbrain.yourdomain.com` |

### 3. How constitution sync works

1. Edit `soul.md` and push to `main`
2. `constitution-sync.yml` triggers automatically
3. It computes the SHA-256 hash of `soul.md`
4. Dispatches `constitution_update` event to all 7 live colonies
5. Each colony's `constitution-receive.yml` workflow downloads the new `soul.md` from this repo and commits it
6. The workflow then commits the updated hash back to `hive.yml`

### 4. Manual triggers

- **Force constitution sync**: Go to Actions → Constitution Sync → Run workflow
- **Check hive health**: Go to Actions → Hive Health Monitor → Run workflow, then check the Summary tab

## Colony Standard Layer

All live colonies expose these endpoints:

```
GET  /colony/info      — colony identity (id, name, role, guilds)
GET  /colony/health    — liveness + uptime
GET  /colony/agents    — registered agent list
POST /colony/events    — ingest a hive event
GET  /colony/manifest  — full manifest including endpoint map
```

(aether uses `/api/colony/*` prefix due to Next.js App Router conventions)
