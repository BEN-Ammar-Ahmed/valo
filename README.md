# NOVA Dashboard

A high-fidelity, five-page desktop-shell dashboard UI, extracted from
[NOVA](https://github.com/BEN-Ammar-Ahmed/Nova), Ahmed's personal AI
assistant. This is the dashboard component on its own: a Python/aiohttp
backend streaming real system state over one WebSocket to a browser (or
Tauri desktop app) frontend — widget dashboard, agent status, apps & files,
calendar, and a command palette.

Full write-up of what's real vs. simulated, the desktop-widget app shell,
security model, and edit-mode controls: see `DESIGN_HANDOFF.md` and
`LIVE-INTEGRATION.md`.

## Quick start (demo mode)

No NOVA agent, no personal accounts, no setup required — this shows the
dashboard with clearly-labeled sample data.

```powershell
python -m venv venv
venv\Scripts\pip install -r requirements.txt
$env:NOVA_DASHBOARD_DEMO = "1"
venv\Scripts\python server.py
```

Then open `http://127.0.0.1:8787`. You'll see a **DEMO DATA** badge in the
top bar — the agent status, Spotify now-playing, Obsidian notes/tasks, model
usage, and conversation feed are all curated sample data (see
`demo_data.py`). System stats (psutil), weather (wttr.in), and your actual
installed-app catalog are real regardless of demo mode — they work for
whoever is running it.

## Real mode

```powershell
venv\Scripts\python server.py
```

Without `NOVA_DASHBOARD_DEMO`, every card reports its honest state: "Agent
offline" until something writes to a NOVA-style bridge heartbeat, "Spotify is
closed" until Spotify is actually open, "Not connected" until you set
`OBSIDIAN_VAULT_PATH` in a `.env` file next to `server.py`. It never invents
data — see `feeds.py`.

This repo runs standalone (`server.py`/`feeds.py` auto-detect that there's no
NOVA `agent.py` next door and treat this folder as the project root), but it's
most useful paired with the full [NOVA](https://github.com/BEN-Ammar-Ahmed/Nova)
agent, which is what actually writes the state this dashboard reads.

## Architecture

```
web/index.html      the design shell (template + logic)
web/support.js       runtime that renders the template
server.py            aiohttp: static files + one WebSocket (/ws) + collectors
feeds.py              real-data collectors
demo_data.py          curated sample data for NOVA_DASHBOARD_DEMO=1
actions.py            handlers for actions the shell sends back over the socket
nova_bridge.py         local agent-status/command bridge (vendored standalone copy)
nova-app/              optional Tauri 2 desktop wrapper (Rust)
tests/                 pytest + Node test suites
```

## Tests

```powershell
venv\Scripts\pip install pytest
venv\Scripts\python -m pytest tests -q
```

## Optional: native desktop app

`nova-app/` wraps this UI as a borderless Tauri 2 desktop widget (its own
`.exe`, tray icon, autostart). See `nova-app/README.md`.

```powershell
cd nova-app
npx tauri dev
```
