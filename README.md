# CD Hunt Metadata (`cd-metadata`)

Flat repository of JSON catalog files for the [CD Hunt](https://github.com/dopingus-build/cd-hunt) web application (`https://dopingus-build.github.io/cd-hunt/`).

Each catalog JSON file sits directly at the repository root and conforms to [`catalog.v1.schema.json`](https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json) (`schemaVersion: 1`). [`index.json`](./index.json) lists preset catalogs shown in the app's Catalog Loader.

## Available Catalogs

- [`2026-japan.json`](./2026-japan.json) — 2026 Japan (Osaka) CD Hunt (`53` physical CDs + `2` digital-only releases)
  - GitHub Blob URL: `https://github.com/dopingus-build/cd-metadata/blob/main/2026-japan.json`
  - Raw URL: `https://raw.githubusercontent.com/dopingus-build/cd-metadata/main/2026-japan.json`

## Loading in CD Hunt

Open [`https://dopingus-build.github.io/cd-hunt/`](https://dopingus-build.github.io/cd-hunt/) and either:

- **Pick a preset directly** from the default **Available Catalog Presets** list,
- **Load a repository's catalog list:** Paste `https://github.com/dopingus-build/cd-metadata` (or any public metadata repo URL / `/tree/<branch>`) into the URL input to load its `index.json` presets, or
- **Load or share a specific catalog file:** Paste a file URL such as `https://github.com/dopingus-build/cd-metadata/blob/main/2026-japan.json` (or open a `?catalog=<raw-json-url>` share link).

## Creating Your Own Metadata Repository (`index.json`)

To host your own presets in a fork or separate public GitHub repository, place your `<name>.json` catalog files at the repository root alongside an [`index.json`](./index.json):

```json
{
  "schemaVersion": 1,
  "catalogs": [
    {
      "id": "my-hunt",
      "title": "My CD Hunt",
      "subtitle": "20 physical CDs",
      "file": "my-hunt.json"
    }
  ]
}
```

## Schema & Agent Instructions

- **Schema & Spec:** [`docs/METADATA.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/METADATA.md) and [`schemas/catalog.v1.schema.json`](https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json) in `cd-hunt`.
- **For LLMs & Coding Agents:** Read [`AGENTS.md`](./AGENTS.md) and the full research guide [`docs/AGENT_CATALOG_GUIDE.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/AGENT_CATALOG_GUIDE.md) before creating or editing catalog files.
