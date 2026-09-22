# Copilot Instructions for this repository

This is a **live Home Assistant configuration directory** (not an application
codebase). It is the actual `/config` folder used by a running Home Assistant
instance, checked into git for backup/versioning. There is no build step,
package manager, or test suite — changes take effect when Home Assistant
reloads or restarts.

## Validating changes

- Home Assistant validates YAML config on reload; there is no separate lint
  command in this repo. After editing `configuration.yaml`, `automations.yaml`,
  `scripts.yaml`, `scenes.yaml`, or `template.yaml`, use HA's built-in
  "Check Configuration" (Developer Tools → YAML, or `ha core check` if running
  under the Supervisor/OS) before restarting.
- Reload the specific domain (e.g. Automations, Scripts, Templates) from
  Developer Tools → YAML instead of a full restart when possible.
- `home-assistant.log` / `home-assistant.log.1` in the repo root show the
  most recent runtime errors — check these after a config change to confirm
  it loaded without errors.

## Layout and where things live

- `configuration.yaml` — main entry point; includes `automations.yaml` and
  `scripts.yaml`, and defines an inline `template:` sensor block (Nord Pool
  price sensors) plus `frontend:`, `zeroconf:`, `default_config:`.
- `automations.yaml` / `scripts.yaml` / `scenes.yaml` — flat lists of
  automation/script/scene definitions, normally managed via the HA UI
  (automations use numeric string `id`s like `'1789560440549'`). Prefer
  editing through the same structure/style already present rather than
  hand-rolled YAML shortcuts.
- `blueprints/` — automation/script/template blueprints, split into
  vendored (`homeassistant/` subfolder = stock blueprints from HA core) vs.
  any custom ones. Don't modify vendored blueprints in place.
- `custom_components/` — third-party integrations installed via HACS
  (`hacs`, `dreame_vacuum`, `tapo_control`, `custom_icons`, `home_intercom`)
  plus `spot_scheduler`. **These are gitignored** (not tracked in git) even
  though the folder exists on disk — do not expect `git` history for files
  under here, and avoid committing them.
- `.storage/` — Home Assistant's internal state store (entity/device/area
  registries, lovelace dashboards, HACS data, auth, etc.). Unusually, this
  **is tracked in git** in this repo. Treat it as generated/runtime state,
  not hand-authored config: prefer changing things via the HA UI/API and
  letting HA rewrite these files, rather than hand-editing JSON here. Be
  careful — it includes `auth` and `http.auth`, which are sensitive.
- `secrets.yaml` — referenced via `!secret` from YAML configs; contains only
  key names of interest here (e.g. `some_password`). Never print/log actual
  secret values; add new secrets as additional keys, don't inline credentials
  into other YAML files.
- `www/community/` — HACS-managed frontend Lovelace card JS bundles (e.g.
  `mushroom`, `button-card`, `apexcharts-card`, `card-mod`). These are
  build artifacts fetched by HACS, not hand-written — don't edit them
  directly, they'll be overwritten on the next HACS update.
- `www/floor3d/` — 3D floor plan assets (`.obj`/`.mtl`) used by the
  floor3d-card.
- `tts/` — cached generated TTS audio files (hashed filenames); safe to
  ignore/regenerate, not meant to be edited.
- `deps/`, `backups/`, `home-assistant_v2.db*`, `*.log*` — runtime data,
  gitignored, don't rely on them being in version control.

## Conventions specific to this config

- Automations/scripts created via the UI get an `id` (automations) that is a
  quoted numeric string derived from a timestamp; keep this pattern for new
  entries added by hand so the UI editor continues to manage them correctly.
- The Nord Pool integration is configured via a `config_entry` ID
  (`01M06P1DCNXDYQY4KBZ5PVDZQP`) referenced directly in `configuration.yaml`
  and `automations.yaml` template/service calls — reuse the same ID rather
  than hardcoding a new one when adding related automations/templates.
- `spot_scheduler` (custom component) also persists its own state under
  `.storage/spot_scheduler.schedules.*`.
