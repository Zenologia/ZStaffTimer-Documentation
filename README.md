# ZStaffTime — Admin Guide 🛠️

*Developed by Zenologia as a trial task* [DevRoom](https://discord.gg/M5b6JWcceP)

**Note: This is only public documenation.  The code is private.**

A concise, admin-friendly guide to installing, configuring, and operating ZStaffTime. This document assumes you have the plugin JAR and will run commands on a Paper/Purpur-compatible Minecraft server.

---

## Quick overview ✅

- Purpose: Track staff shifts (clock in/out), measure active/idle/paused time, show leaderboards and a configurable GUI.
- Core systems:
  - SessionManager — live shift tracking, AFK detection, periodic persistence.
  - GUI — MainMenu + WhoMenu (configurable via config.yml).
  - Commands — `/zstafftimer` subcommands.
  - Storage — SQLite (default) or MySQL (optional).

---

## Installation (server operator) 🚀

1. Place the built plugin JAR in `plugins/`.
2. Restart the server. Paper reads `paper-plugin.yml` from inside the JAR on startup — command registrations are read then.
3. After first enable the plugin will create `plugins/ZStaffTime/config.yml`. Edit this file to customize behavior and UI.
4. Use `/zstafftimer` in-game. Admins use subcommands described below.

Note: Storage backend changes (SQLite ↔ MySQL) require a full server restart, not just reload.

---

## Permissions (default nodes) 🔐

- `zstafftimer.tracked` — Tracked staff (base access & auto-tracking). Default: false.
- `zstafftimer.management` — Management features (who/history for others, GUI manager). Default: false.
- `zstafftimer.admin` — Admin actions (adjust, reload). Default: false.
- `zstafftimer.leaderboard` — Leaderboard access. Default: false.

Grant these via your permissions plugin (LuckPerms, etc.).

---

## Commands (admin-oriented) 📋

- `/zstafftimer` — Open GUI (players) or show help (console).
- Subcommands:
  - `/zstafftimer clockin` — Start your shift.
  - `/zstafftimer clockout` — End your shift.
  - `/zstafftimer status` — Show your current shift state and times.
  - `/zstafftimer addnote <message...>` — Add a note to your active shift.
  - `/zstafftimer history [page]` — View recent shifts (management can specify a player).
  - `/zstafftimer who` — Print currently clocked-in staff (management).
  - `/zstafftimer leaderboard` — Show top active-time leaderboard.
  - `/zstafftimer adjust <player> <active|idle> <+/-seconds> <reason...>` — Admin: adjust a player's most recent shift totals.
  - `/zstafftimer reload` — Reloads config/messages (requires `zstafftimer.admin`).

---

## Config overview (what to edit) ⚙️

Location: `plugins/ZStaffTime/config.yml`

Key sections you'll use:

- settings
  - `settings.afk_timer_seconds` — seconds before marking idle.
  - `settings.history.last_n_shifts` — page size for history output.

- database
  - `database.type` — `SQLITE` (default) or `MYSQL`.
  - SQLite: `database.sqlite.file`
  - MySQL: `database.mysql.host`, `.port`, `.database`, `.username`, `.password`, `.pool.*`

- ui
  - `ui.main_menu.title` — inventory title.
  - `ui.main_menu.size` — inventory size (27, 54 etc).
  - `ui.main_menu.items.<item>` — each main-menu item has `slot`, `material`, `name`, and `lore`.
    - Supported items (defaults present): `my_shift`, `clock_in`, `add_note`, `clock_out`, `history`, `who`, `leaderboard`, `adjust_help`.
    - Example (Clock In):
      ```yaml
      ui:
        main_menu:
          items:
            clock_in:
              slot: 8
              material: "EMERALD"
              name: "<green><bold>Clock In</bold></green>"
              lore: ["<gray>Click to start your shift.</gray>"]
      ```
    - Slots use inventory indexes (0..size-1). Change slots + reload to reposition.
  - `ui.who_menu` — who-list UI templates and `controls.back.slot`.

- messages
  - `messages.*` — all user-facing strings. Ensure keys like `messages.reloaded` exist to avoid warnings.

---

## GUI behavior & config details 🎛️

- The Main Menu reads `ui.main_menu.items.<item>.slot` and places items accordingly — this lets admins rearrange the layout without code changes.
- The Clock In item is configurable like other items: `slot`, `material`, `name`, `lore`.
- Click handling reads the slots from config at click-time, so updating the config and running `/zstafftimer reload` will update behavior immediately.
- The Who menu lists active sessions (vanish filtering was removed in the current plugin state). The "Back" control slot is `ui.who_menu.controls.back.slot`.

---

## Storage & persistence 🗄️

- Default: SQLite file at `database.sqlite.file` (`stafftime.db`).
- MySQL: configure `database.type: "MYSQL"` and supply `database.mysql.*` settings.
- The plugin periodically persists open shifts (every ~30 seconds) and runs crash-recovery on startup to close dangling shifts.
- Do not change storage backend while server is running — restart required.

---

## AFK handling & autosave ⏱️

- AFK detection parameter: `settings.afk_timer_seconds` (affects idle status).
- SessionManager runs:
  - Async periodic flush to storage (every ~30 seconds).
  - Sync AFK checks/warnings (every second).
- Use `/zstafftimer status` to show active/idle/paused totals.

---

## Common troubleshooting 🐞

- Plugin not loaded / disabled on startup:
  - Check `/plugins` (or `/pl`) to confirm ZStaffTimer is enabled (not red/disabled).
  - Check server logs on restart: plugin prints "ZStaffTime enabled." and may log storage initialization messages.
  - If plugin failed to enable, paste the enable-time stacktrace for diagnosis.

- `Missing messages.reloaded in config.yml`:
  - Add `messages.reloaded: "{prefix}<green>Configuration reloaded.</green>"` to `messages` in config.yml.

- Invalid Material in config:
  - Use Material names supported by your server version (case-insensitive). The code falls back to defaults if invalid.

- Storage initialization failure:
  - If using MySQL, check credentials and network. For SQLite, check file permissions. Inspect server logs for StorageInitException details.

---

## Best practices & admin tips ✅

- Keep `messages.reloaded` and other keys present to avoid warnings on reload.
- Use config-driven slots to create a consistent UI across servers and server networks.
- Prefer the `adjust` command to correct shift totals rather than editing DB directly.
- For multi-server deployments, use a shared MySQL backend and consistent timezone settings if you aggregate leaderboards.

---

## Example snippets (quick reference) 📌

Add configurable Clock In item:
```yaml
ui:
  main_menu:
    items:
      clock_in:
        slot: 8
        material: "EMERALD"
        name: "<green><bold>Clock In</bold></green>"
        lore:
          - "<gray>Click to start your shift.</gray>"
```

Add a reload message:
```yaml
messages:
  reloaded: "{prefix}<green>Configuration reloaded.</green>"
```

---

## Appendix — quick keymap 🔑

- Config: `plugins/ZStaffTime/config.yml`
- Main Menu slots: `ui.main_menu.items.<item>.slot`
- Clock-in: `ui.main_menu.items.clock_in.{slot,material,name,lore}`
- Who menu back: `ui.who_menu.controls.back.slot`
- Storage: `database.type` (`SQLITE` or `MYSQL`)

---

## 🧑‍💻 Author

- **Zenologia**
