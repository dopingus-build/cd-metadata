# CD Hunt Metadata (`cd-metadata`)

Flat repository of JSON catalog files for the [CD Hunt](https://github.com/dopingus-build/cd-hunt) web application (`https://dopingus-build.github.io/cd-hunt/`).

Each catalog JSON file sits directly at the repository root and conforms to [`catalog.v1.schema.json`](https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json) (`schemaVersion: 1`). [`index.json`](./index.json) lists preset catalogs shown in the app's Catalog Loader.

## Available Catalogs

- [`2026-japan.json`](./2026-japan.json) — Osaka CD Cover Album & Checklist (53 physical CDs + 2 digital-only releases)
  - Raw URL: `https://raw.githubusercontent.com/dopingus-build/cd-metadata/main/2026-japan.json`

## Schema & Agent Instructions

- **Schema & Spec:** [`docs/METADATA.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/METADATA.md) and [`schemas/catalog.v1.schema.json`](https://dopingus-build.github.io/cd-hunt/schemas/catalog.v1.schema.json) in `cd-hunt`.
- **For LLMs & Coding Agents:** Read [`AGENTS.md`](./AGENTS.md) and the full research guide [`docs/AGENT_CATALOG_GUIDE.md`](https://github.com/dopingus-build/cd-hunt/blob/main/docs/AGENT_CATALOG_GUIDE.md) before creating or editing catalog files.
