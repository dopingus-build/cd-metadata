# AGENTS.md — Working with `cd-metadata`

This repository holds flat, root-level JSON catalog files for the [**CD Hunt**](https://dopingus-build.github.io/cd-hunt/) web application ([`dopingus-build/cd-hunt`](https://github.com/dopingus-build/cd-hunt)).

Before creating a new catalog or adding/updating items in an existing one, read the full research playbook in `cd-hunt`:
- **Agent Catalog Guide:** [`docs/AGENT_CATALOG_GUIDE.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/AGENT_CATALOG_GUIDE.md) (local sibling: `../cd-hunt/docs/AGENT_CATALOG_GUIDE.md`)
- **Metadata Spec:** [`docs/METADATA.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/METADATA.md)
- **JSON Schema (`v1`):** [`catalog.v1.schema.json`](https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json)

---

## Repository Layout Rules

1. **Keep the repository flat.** Place every catalog JSON file directly at the repository root (e.g., `2026-japan.json`, `vocaloid-essentials.json`). Never nest catalogs inside subdirectories.
2. **Register preset catalogs in `index.json`.** Whenever you add a new root-level catalog file that should appear in the app's **Preset Catalogs** picker, add an object to `index.json` under `catalogs` with:
   - `id`: short URL-safe slug (e.g., `"2026-japan"`)
   - `title`: display title in the picker
   - `subtitle`: concise 1-line summary (item count / scope)
   - `file`: root filename (e.g., `"2026-japan.json"`)

---

## Critical Editing & Schema Constraints

- **Never rename, renumber, or reuse existing `item.id` values.**
  User checkmarks, notes, and cached covers in `localStorage` are keyed by `item.id`. When inserting into an existing catalog, append the next unused sequential ID (e.g., `cd-53`, `dig-03`) or add a suffix (`cd-11b`). IDs must match `/^[a-zA-Z0-9_-]+$/`.
- **Always include `schemaVersion: 1` (and recommended `$schema`):**
  ```json
  "$schema": "https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json",
  "schemaVersion": 1
  ```
- **Never set `meta.storagePrefix`.**
  `cd-hunt` computes storage prefixes automatically from the URL/filename/metadata (`js/storage.js`) and explicitly rejects `meta.storagePrefix`. (`meta` only allows `title`, `subtitle`, `documentTitle`, `searchPlaceholder`, `digitalSectionTitle`, and `digitalSectionSubtitle`.)
- **Verify physical CD vs. digital-only (`digitalOnly`):**
  Set `"digitalOnly": true` (`catalog: "Digital Only"`, `regCatNo: "DIGITAL"`, `mbEditions: []`) for streaming/download-only tracks; omit or set `"digitalOnly": false` for physical pressings.
- **Keep categories consistent:**
  If `categories` is present, it MUST include `"key": "All"`, and every item's `category` MUST match one of `categories[].key`.
- **Prioritize Regular Editions (`通常盤`):**
  Set `regCatNo` to the Regular Edition catalog number and put `"通常盤 Regular"` (containing `"通常"`) first in `mbEditions`.
- **Formatting conventions for UI & SVG rendering:**
  - `price`: place the yen range **before** the first `(` (e.g., `"¥500–¥1,100 (Reg)"`), because album cards render `price.split('(')[0].trim()`.
  - `kana`: keep the segment before `/` concise (e.g., `"やま / や行 (Yama)"`), as it renders on the fallback SVG jewel-case obi.
  - `requiredGroups`: required whenever `itunesQuery` is non-empty! Use a non-empty `string[][]` representing AND-of-ORs case-insensitive substring guards matched against `trackName + ' ' + collectionName + ' ' + artistName`.
- **No prohibited domains anywhere in any JSON string:**
  Never include `vgmdb.net`, `wikia.nocookie.net`, `fandom.com`, or `cdjapan.co.jp`. Cover art is resolved dynamically in the browser via MusicBrainz/Cover Art Archive and iTunes Search API.

---

## Live API Verification Commands

Verify catalog numbers and search guards before committing new or modified entries:

```bash
# 1. Verify MusicBrainz release MBID + Cover Art Archive front-500 status
CATNO="SRCL-12539"
MBID=$(curl -fsSL -A "cd-metadata-verifier/1.0 (https://github.com/dopingus-build/cd-metadata)" \
  "https://musicbrainz.org/ws/2/release/?query=catno:${CATNO}&fmt=json&limit=4" \
  | node -e '
    const d = JSON.parse(require("fs").readFileSync(0, "utf8"));
    for (const r of (d.releases || [])) {
      console.error(r.id, "|", r.title);
    }
    if (d.releases?.[0]?.id) process.stdout.write(d.releases[0].id);
  ')
[ -n "$MBID" ] && curl -sS -o /dev/null -w "CAA status: %{http_code}\n" -L "https://coverartarchive.org/release/${MBID}/front-500"

# 2. Verify Apple iTunes Search API + requiredGroups filter (country=jp and country=us)
curl -fsSL "https://itunes.apple.com/search?term=slash%20yama&country=jp&entity=song&limit=8" \
  | node -e '
    const groups = [["slash"], ["yama"]];
    const d = JSON.parse(require("fs").readFileSync(0, "utf8"));
    for (const r of (d.results || [])) {
      const hay = `${r.trackName || ""} ${r.collectionName || ""} ${r.artistName || ""}`.toLowerCase();
      const ok = groups.every(g => g.some(tok => hay.includes(tok.toLowerCase())));
      console.log(ok ? "MATCH" : "SKIP ", "|", r.artistName, "|", r.trackName, "|", r.collectionName);
    }
  '
```

---

## Validation Before Commit & Push

### Option A: With sibling `../cd-hunt` repository present (Recommended)

```bash
# Validate a single catalog with cd-hunt's runtime validator
node -e '
  const fs = require("fs");
  const { validateCatalog } = require("../cd-hunt/js/validate.js");
  const data = JSON.parse(fs.readFileSync("./2026-japan.json", "utf8"));
  const res = validateCatalog(data);
  if (!res.valid) { console.error(res.errors); process.exit(1); }
  console.log("VALID:", data.items.length, "items");
'

# Run the full cd-hunt unit + integration suite (auto-validates ../cd-metadata/*.json & index.json)
node --test ../cd-hunt/test/*.test.js
```

### Option B: Standalone JSON & Schema Check (without local `../cd-hunt`)

```bash
# Verify JSON syntax on all root files and inspect against the published schema URL
node -e '
  const fs = require("fs");
  const index = JSON.parse(fs.readFileSync("./index.json", "utf8"));
  for (const entry of index.catalogs) {
    const c = JSON.parse(fs.readFileSync(`./${entry.file}`, "utf8"));
    if (c.schemaVersion !== 1 || !c.$schema || !Array.isArray(c.items)) {
      throw new Error(`Invalid header in ${entry.file}`);
    }
    if (c.meta && "storagePrefix" in c.meta) {
      throw new Error(`meta.storagePrefix is forbidden in ${entry.file}`);
    }
    console.log("OK:", entry.file, "items:", c.items.length);
  }
'
```
