# OK-OFFLINE 2026 release ledger

This is the operational authority for **v4.0.0 — OK-OFFLINE — Black Rock City Guide** and its location-release updates. The application is not affiliated, endorsed, or verified by Burning Man Project.

## Release sequence

| Version | Earliest public time | Public data |
| --- | --- | --- |
| 4.0.0 | now | 2026 camp, art, and event content; no camp or art placement |
| 4.0.1 | 2026-08-23 00:00 America/Los_Angeles | camp placement; art placement remains absent |
| 4.0.2 | 2026-08-30 00:00 America/Los_Angeles | camp and art placement; final event refresh |

The August 9 developer release is validation-only. It may retain count-only output for private review and must not publish or retain raw location-bearing API responses.

## Current authoritative snapshot

Fetched and sanitized on 2026-07-20:

- 1,201 camps
- 323 art pieces
- 2,231 API event rows
- 2,205 published unique events
- 26 byte-identical duplicate event rows removed
- 0 divergent duplicate UIDs
- 1 unresolved camp reference, `a1XVI00000FJ1B32AL`, retained as an unlinked event warning
- 0 camp locations, 0 art locations, and 0 derived placement fields published
- no official 2026 OKNOTOK camp record

This explains the change from the earlier planning observation of 2,233 event rows, 2,217 unique events, and 16 duplicates. The source changed; the validated current snapshot therefore contains **3,729**, not 3,741, searchable UIDs.

The GIS source is pinned to `burningmantech/innovate-GIS-data` revision `3c69f43ef58a55f6cb5f5c6c99a64f4ec5966145`. All eight official 2026 layers are present: city blocks, CPNs, DMZ, plazas, street lines, street outlines, toilets, and trash fence.

## Implemented release boundary

### PWA

- One canonical season registry controls years, dates, timezone, avenue names, release clocks, GIS revisions, map landmarks, safety floors, and current defaults.
- Existing `bm2025-db` IndexedDB data and all user content remain in place; the 2026 default is applied once and historical years remain selectable.
- `scripts/sync-season-data.js` is fail-closed, stage-and-atomic, schema-strict, duplicate-aware, reference-aware, hash-producing, and embargo-enforcing.
- Public data is network-first with validated cached fallback. Immutable revisioned build assets are cache-first. The service-worker identity is version plus build revision.
- Routing uses the pinned 2026 street geometry and retains the historical 2025 adapter.
- Weather credentials and provider calls exist only in Rails; the PWA keeps a clearly aged seven-day last-success fallback.
- CI runs unit, build, and real-browser offline/install/update checks. Promotion accepts an exact main SHA and targets immutable release directories.

### Rails

- `CURRENT_BURN_YEAR` defaults to 2026 and the data root defaults to the sibling PWA snapshot.
- Search import is split into metadata, embeddings, and structured entities. It upserts by authoritative UID, deletes only absent records for that year/type, and retains vectors when searchable text is unchanged.
- The embedding invariant remains `text-embedding-3-small` with 1,536 dimensions.
- Status and analytics do not construct an LLM client. Vector unavailability is a typed 503 and the PWA falls back to local keyword search.
- OpenWeather is primary, WeatherKit is fallback, and the last successful response can be served as labeled stale data.
- A promotion backup contains both a full custom-format PostgreSQL dump and the exact sanitized source snapshot. Rollback is guarded by database-name confirmation and requires the API listener to be stopped.

## Promotion gates

Do not promote because source exists. Record each boundary separately:

1. PWA tests, production build, data verification, and browser smoke are green.
2. Rails tests, Zeitwerk, RuboCop, and Brakeman complete without scanner errors.
3. A fresh database backup and snapshot checksum are recorded.
4. 2026 search count equals snapshot published counts; embedding coverage is 100%; repeated metadata import preserves every vector.
5. Structured output parses through `BasicEntityExtraction`; semantic and hybrid queries return 2026 results.
6. Search metadata, IndexedDB, direct JSON URLs, and rendered UI contain no data beyond the active location phase.
7. Live primary weather, forced WeatherKit fallback, stale cached response, and age labeling are proven.
8. Rails code is active and `/up`, search status, semantic/hybrid search, and weather pass through `https://offline.oknotok.com`.
9. Caddy serves `/var/www/offline.oknotok.com/current`; the GitHub `production` environment requires approval; deployment secrets are current.
10. Promote the exact PWA SHA and verify release identity, JSON content types/counts, manifest installability, service-worker replacement, and offline reload through the public domain.

## Recovery

Before every search promotion, run:

```bash
ok-offline-api/bin/backup-search-promotion 2026
```

To restore a recorded recovery point, stop `app.oknotok.ok-offline-api`, then supply the exact database name as the confirmation value:

```bash
CONFIRM_DATABASE_RESTORE=ok_offline_api_development \
  ok-offline-api/bin/rollback-search-promotion \
  ok-offline-api/storage/search-backups/2026/<timestamp>
```

Restart Rails, verify the matching sanitized snapshot and search counts, then switch the static symlink to the corresponding immutable release. Keep the previous two static releases and their database/snapshot recovery points.

## Remaining external gates as of 2026-07-20

- OpenWeather primary, forced WeatherKit fallback, and labeled stale-cache behavior have live proof through Rails. Provider credentials remain server-only.
- Rails search status, semantic/hybrid search, and weather pass through `https://offline.oknotok.com`.
- The public static host is still the old release: `/data/2026/*.json` currently resolves to an HTML SPA fallback and the manifest still uses the old installed-app name.
- The Caddy document root has not yet been proven to point at the new `current` symlink, and `/up` is not yet proven to reach Rails through the public domain.
- GitHub environment approval and deployment secrets require repository-side verification.
- Source changes are local and uncommitted; no release tag or public promotion has occurred.
