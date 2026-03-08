# ExoScanner Mod Overview

This README is a practical reference for features, limitations, and file locations based on the current codebase.

## Feature Overview

1. Local (Client-Side) Armor Scanning
   - Scans armor worn by players in the current client world.
   - Applies whitelist rules and illegal-color detection.
   - Stores results in the local JSONL database.
   - Triggers notifications if enabled.
   - Seymour items are force-scanned and tiered.
   - Runs automatically on a configurable interval and can be run immediately via command.

2. Hypixel API Scanning
   - Uses Mojang + Hypixel APIs to scan player inventories/wardrobes.
   - Supports multiple API keys with cooldown/rate tracking.
   - Can scan a single player or an entire lobby via API.
   - Stores results in the same local database.

3. Auto Lobby Scan
   - Optional background scan on lobby switch and/or periodic interval.
   - Debounces scans and enforces minimum gaps between API pulls.
   - Uses per-key usage tracking to avoid hammering API keys.

4. Player Tracing (Client-Side Only)
   - Manual tracer: `/exo trace <player>` draws a line to a selected player.
   - Auto tracer: configurable list of usernames to trace when found.
   - Max auto targets configurable (1–5).
   - Auto-disables when within a configurable radius.
   - Line width, color, and vertical offset are configurable.
   - If a player isn’t found, it stops and prints a chat failure message.
   - Completely client-side; sends no packets.

5. Database
   - JSONL file storage per armor set.
   - De-duplication by armor UUID when available; fallback to soft key (owner + armor + color).
   - Soft-key entries are replaced automatically if a UUID version later appears.

6. Search / Query
   - `/exo search <armor|ANY> <color> [flags]`
   - `/exo search player <username>`
   - Supports color distance (`de=`), time filters (`since=`), and metric selection.
   - Suggestions are cached to avoid freeze when typing.

7. Exports
   - Export database to CSV or XLSX via `/exo export csv` or `/exo export xlsx`.
   - Supports filters for time range, armor, and illegal type.

8. Inventory Highlight
   - Optional overlay/highlight style on inventory slots (configurable).

9. Debugging / Logging
   - Separate toggles for armor scan, API scan, lobby scan, and tracer debug.
   - Missing UUID debug for armor scans (with optional chat notification).

## What Works

- Local scanning logic and whitelist filtering.
- Hypixel API scanning with key rotation and rate tracking.
- Database storage and de-duplication logic.
- Searches and exports from the local DB.
- Player tracer feature (manual + auto), including width/color/offset config.
- Config UI and persistence.
- Auto lobby scan timing controls.

## What Does Not Work (Known Limitation)

1. Non-API Armor UUID Extraction
   - For many pieces, the client does not receive the UUID in NBT.
   - Example logs show only `customComponent={id:"TARANTULA_HELMET"}` with no UUID data.
   - This is a data availability limitation, not a parsing bug.
   - Result: local scans can’t produce UUIDs for those pieces, so dedupe uses soft keys.

If desired, missing-UUID chat notifications can be suppressed when only `{id:...}` is present to reduce spam.

## File Locations

All paths are resolved relative to Fabric’s config directory unless otherwise specified.

### Config Directory (Windows typical)
- Vanilla: `C:\Users\<you>\AppData\Roaming\.minecraft\config`
- Modrinth profiles: `C:\Users\<you>\AppData\Roaming\ModrinthApp\profiles\<profile>\config`

### Files
- Main config: `exoscanner.json`
- Whitelist: `exoticarmorwhitelist.json`
- Scan cache: `exoscanner_cache.json`
- DB directory (default): `exoscanner_db\`
- Default resources: `colors.json`, `checklistdata.json`

### Database Files
- Stored as JSONL in: `<configDir>\exoscanner_db\`
- Each file corresponds to an armor set (for example `NECRON.jsonl`).
- If `dbPath` is set in config:
  - Absolute paths are used directly.
  - Relative paths are resolved under the config directory.

### Export Files
- Written into the config directory:
  - `exo_export_<timestamp>.csv`
  - `exo_export_<timestamp>.xlsx`
