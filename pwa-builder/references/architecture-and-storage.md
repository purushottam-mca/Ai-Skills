# Architecture and Storage Reference

Use this reference when a PWA request requires a concrete data model, storage choice, migration plan, export/import design, service-worker strategy, or deployment decision. Keep the core skill’s simplicity rule in force.

## Contents

1. [Architecture worksheet](#architecture-worksheet)
2. [Storage selection](#storage-selection)
3. [IndexedDB repository pattern](#indexeddb-repository-pattern)
4. [Schema and migration strategy](#schema-and-migration-strategy)
5. [Export and import](#export-and-import)
6. [Backup and recovery](#backup-and-recovery)
7. [Service-worker strategy](#service-worker-strategy)
8. [Online boundaries](#online-boundaries)
9. [Deployment](#deployment)

## Architecture worksheet

Before coding, complete a short decision record. Keep it in the project when the app is large enough to benefit from traceability.

| Question | Decision to record |
| --- | --- |
| Primary user and task | Who is using the app, and what should become easier? |
| Smallest useful release | What can be removed without invalidating the product? |
| Offline contract | Which features must work without network access? |
| Local data | What records, settings, drafts, files, or queues exist? |
| Online data | What is remote, why is it remote, and what happens when unavailable? |
| Source of truth | Browser-local data, remote data, or an explicitly defined hybrid? |
| Recovery | How can the user undo, export, restore, or recover from failure? |
| Privacy boundary | What data leaves the device, and which third parties receive it? |
| Complexity additions | Which dependency or service is being added, and what concrete requirement pays for it? |

## Storage selection

Use the smallest mechanism that satisfies the requirement.

| Need | Preferred mechanism | Design note |
| --- | --- | --- |
| Theme or display preference | `localStorage` | Store a small JSON-safe value; tolerate missing or corrupt preference data. |
| A collection of records | IndexedDB | Use a repository with typed operations and explicit schema versioning. |
| Search or indexes over records | IndexedDB object stores and indexes | Keep queries purposeful; do not build a general query engine. |
| App shell and static assets | Cache API via service worker | Version caches and remove old versions during activation. |
| Offline queue for a remote action | IndexedDB | Store operation ID, payload, state, retry metadata, and error detail. |
| Large local file workspace | OPFS, where supported and justified | Treat browser support, quota, and user backup as product concerns. |
| Sensitive credential | Do not use browser storage by default | Prefer a secure authenticated flow; never treat local storage as a secret vault. |
| User backup | User-visible file outside browser-managed storage | Use cloud storage only as an optional backup destination; do not confuse OPFS with an external backup. |

Use `localStorage` only for small, non-critical preferences. It is synchronous and not a suitable primary database for user collections. Do not store a growing array of notes, transactions, bookmarks, or game state in one key.

Use IndexedDB for structured data. Keep the API behind a repository, for example:

```ts
export type RecordId = string;

export interface Entry {
  id: RecordId;
  createdAt: string;
  updatedAt: string;
  title: string;
  body: string;
}

export interface EntryRepository {
  list(): Promise<Entry[]>;
  get(id: RecordId): Promise<Entry | undefined>;
  put(entry: Entry): Promise<void>;
  remove(id: RecordId): Promise<void>;
}
```

The UI should depend on `EntryRepository`, not on `IDBObjectStore`, transaction lifecycles, or storage key names. This keeps a small app understandable while leaving room to test the domain logic without a browser.

## IndexedDB repository pattern

Create one database with a clear name and an explicit version. Use one object store per meaningful entity or bounded collection. Define indexes only for actual access patterns, such as `updatedAt`, `category`, or a normalized search key. Avoid speculative indexes.

Use transactions that match the operation. Keep transaction scopes short, validate data before opening a write transaction, and treat `QuotaExceededError`, blocked upgrades, and browser storage failures as user-visible recoverable conditions. Never interpret an unavailable database as an empty database.

Generate stable random identifiers rather than relying on timestamps alone. Preserve `createdAt` separately from `updatedAt`. Store dates in a consistent serializable format, usually ISO strings, and define timezone behavior explicitly for date-only values such as birthdays or billing periods.

Keep derived values derivable. For a finance tracker, totals can normally be calculated from transactions; for notes, a search index can be regenerated; for a game, a compact state snapshot may be persisted when it materially improves resume behavior. If derived data is cached for performance, define how it is invalidated and rebuilt.

## Schema and migration strategy

Start with a schema record even for version one:

```ts
export const DB_NAME = "personal-app";
export const DB_VERSION = 1;

export type ExportEnvelope = {
  format: "personal-app-export";
  app: string;
  schemaVersion: number;
  exportedAt: string;
  data: unknown;
};
```

For each database version, document the stores, indexes, field meaning, and migration steps. Migrations should be deterministic and ordered. A later release should be able to open data from every supported prior version or clearly explain the unsupported case before making changes.

Follow this upgrade sequence:

1. Detect the current schema version.
2. Preserve the existing database until the next version is ready.
3. Create or alter stores and indexes in the upgrade transaction.
4. Transform records conservatively, preserving unknown data where safe.
5. Record the new version only after the transformation succeeds.
6. Run a post-migration integrity check.
7. Surface a recovery state if any step fails.

Test migrations with fixtures from every supported version, empty databases, partially populated databases, malformed records, and duplicate or missing identifiers. Include a backup/export path before high-risk migrations.

## Export and import

For important local data, provide a visible **Export data** action and a straightforward **Import data** action. Export should be usable without an account or network connection.

Use a versioned envelope rather than exporting an unlabelled array:

```json
{
  "format": "personal-app-export",
  "app": "example-notes",
  "schemaVersion": 2,
  "exportedAt": "2026-08-19T12:00:00.000Z",
  "data": {
    "notes": []
  }
}
```

On import, perform these checks before writing anything:

| Check | Required behavior |
| --- | --- |
| File type and size | Reject unsupported or unreasonably large input with an understandable message. |
| Parse | Catch invalid JSON or malformed binary content without crashing the app. |
| Envelope | Verify app identifier, format, and schema version. |
| Shape | Validate required fields, identifiers, dates, and collection types. |
| Compatibility | Migrate older supported exports in memory before persistence. |
| Collision policy | Explain merge, replace, skip, or duplicate behavior before committing. |
| Recovery | Offer or recommend exporting current data before replacement. |

Never render imported HTML directly. Treat imported text as data and escape it in the UI. If binary files are included, define whether the export is a ZIP, a manifest plus files, or only metadata; do not imply that a JSON export contains files it cannot contain.

## Backup and recovery

For valuable personal data, distinguish three layers:

```text
1. Working copy
   IndexedDB / Dexie

2. Browser resilience
   navigator.storage.persist() when granted

3. Recovery copy
   user-visible .json or encrypted .enc file
   stored outside browser-managed origin storage
```

`navigator.storage.persist()` reduces the chance of automatic eviction under storage pressure when granted, but it does not prevent a user from explicitly clearing site/app data. Treat that distinction as a product requirement, not a footnote.

Do not use OPFS as the sole disaster-recovery backup. OPFS is still origin-private, browser-managed storage. Use a user-visible file chosen through download/export or a user-authorized filesystem picker when the goal is recovery after browser data deletion.

For a local-first app without synchronization requirements, a simple export/restore flow is usually preferable to adding a backend. If automatic off-device backup is later desired, add a backup-provider abstraction:

```text
BackupManager
  ├── LocalFile / Download
  ├── Google Drive (optional)
  ├── Proton Drive (optional)
  ├── Ente / WebDAV / other provider (optional)
  └── Future providers
```

The core app should only depend on `BackupManager`, not a specific cloud provider. Keep backup upload separate from the local source of truth. A failed cloud upload must never make a successful local write appear unsaved.

If encryption is enabled, encrypt the export before sending it anywhere. Do not rely on provider-side encryption alone when the product requirement is that the provider cannot read the user's backup. Prefer vetted authenticated encryption (for example AES-GCM via Web Crypto) and a vetted password-based KDF; never invent cryptography.

Backup reminders should normally be based on **time since last successful backup and/or amount of meaningful new data**, not a forced daily interruption.

## Service-worker strategy

Keep the service worker narrow and explicit. A typical small app needs:

```text
install  -> precache the app shell and immutable build assets
activate -> remove old caches and claim clients only if the update policy is intentional
fetch    -> apply route-specific strategies; never cache blindly
message  -> optionally coordinate a user-approved update
```

Name caches by purpose and build version, for example `app-shell-v3` and `runtime-images-v2`. Precache only stable build output. Do not hand-maintain a list that the build system can invalidate incorrectly; use the chosen toolchain’s manifest support when it is reliable.

Use these strategies deliberately:

| Request | Typical strategy | Reason |
| --- | --- | --- |
| Hashed JS/CSS/image asset | Cache-first | The filename changes when content changes. |
| Navigation shell | Network-first with cached fallback, or precache-first for a fully local app | Preserve a usable app during network failure. |
| Public content that tolerates staleness | Stale-while-revalidate | Display cached content quickly and refresh in the background. |
| Private/authenticated response | Network-only unless a security review says otherwise | Avoid leaking sensitive data through shared caches. |
| User data write | Do not treat Cache API as the database | Persist in IndexedDB or send through an explicit online boundary. |

Test the update lifecycle rather than assuming it works. Verify old caches are removed, the app remains usable during an update, an edit is not lost, and an update prompt cannot cause an infinite reload loop.

## Online boundaries

Represent remote capabilities as explicit domain operations. For each one, define local behavior, pending behavior, retry behavior, and permanent failure behavior.

```text
local edit -> persist locally -> mark pending if remote sync exists
             -> attempt remote operation when allowed
             -> acknowledge, retry with backoff, or surface a resolvable conflict
```

Do not use the browser’s `navigator.onLine` value as proof of server availability. Use it to adjust hints and retry timing, then rely on actual request outcomes. Do not block local reads or local edits while waiting for a remote request.

If synchronization is introduced, document conflict semantics before implementation. “Last write wins” may be acceptable for a low-value personal preference but is dangerous for financial records or collaborative notes. Include operation IDs or revision metadata so retries do not duplicate effects.

## Deployment

Prefer a static host or simple self-hosting with HTTPS. Document the build command, output directory, required headers, and any base-path configuration. If the app is hosted below a subpath, test the manifest `start_url`, asset URLs, service-worker scope, and router fallback in that subpath.

Release with a cache/version update plan. Verify that a newly deployed build eventually reaches an existing installed client, that old assets do not break a new service worker, and that users have a clear update path. Keep rollback possible: a bad service-worker release can persist beyond an ordinary page refresh.

For self-hosting, minimize infrastructure. A static file server plus correct MIME types, HTTPS, and fallback behavior is usually sufficient for a local-first PWA. Add a backend, reverse proxy, database, or container only when the application’s requirements make it necessary.

