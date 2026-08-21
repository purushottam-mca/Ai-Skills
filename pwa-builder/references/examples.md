# Practical PWA Guidance Examples

Use these examples to calibrate architecture and product decisions. They are patterns, not rigid templates. Keep the same reasoning order for a different app: clarify the task, define the offline contract, model data, choose storage, design the primary flow, then add only justified capabilities.

## Personal finance tracker

### Product decision

A personal finance tracker is a strong fit for an offline-first PWA when the user wants private, device-local records and is willing to manage backup or synchronization deliberately. Do not add a bank connection, accounts, or a server unless the user explicitly needs transaction import, cross-device sync, shared finances, or institution-specific integration.

### Default architecture

Use a static app with a small domain layer and IndexedDB. Store accounts, categories, transactions, and app preferences as separate stores or clearly separated collections. Store amounts as integer minor units, such as cents, rather than binary floating-point numbers. Keep currency and sign conventions explicit. Derive balances and summaries from transactions, with optional cached aggregates only when measurements show a real performance need.

```text
Account:    id, name, currency, openingBalance, archivedAt?
Category:   id, name, kind, colorToken?
Transaction: id, accountId, categoryId?, amountMinor, currency,
             occurredOn, payee?, note?, createdAt, updatedAt
Preference: theme, defaultAccountId, defaultDateRange
```

### UI guidance

Make “Add transaction” the primary action. The first screen should show a useful balance or recent activity, not a dashboard of every possible metric. Use a focused transaction form with date, amount, account, and optional category/note. Let the user save locally immediately. Provide filters and summaries as secondary views; avoid turning the app into a spreadsheet unless bulk reconciliation is the actual requirement.

### Safety and backup

Explain that data is stored locally on this device. Treat IndexedDB/Dexie as the working source of truth, not as an indestructible backup. Provide export/import prominently enough to discover, validate imports before replacement, and recommend an export before destructive migration or replace-mode import. If the user is unlikely to use manual exports, a non-blocking stale-backup reminder can help. Never claim that local data is synchronized or backed up automatically unless that behavior exists. If encrypted backups are offered, encrypt them client-side before saving or uploading and make password loss/recovery behavior explicit.

### Tests

Test decimal and negative-value handling, date-only behavior, category deletion, archived accounts, balance derivation, duplicate import handling, export round-trips, empty states, refresh persistence, offline reload, and recovery after a failed write. Include a migration fixture before changing the transaction schema.

## Bookmark manager

### Product decision

A local bookmark manager is a strong PWA candidate. The core value—saving, organizing, searching, and opening URLs—works without a backend. Remote page metadata, favicon retrieval, broken-link checks, and synchronization are optional online capabilities and should not block local bookmark management.

### Default architecture

Use IndexedDB for bookmarks and tags, with indexes or normalized search fields for title, URL, and updated time. Use `localStorage` only for view preferences such as sort order or the last selected tag. Treat fetched titles, descriptions, and favicons as best-effort metadata; store the user-entered URL as the authoritative value.

```text
Bookmark: id, url, title, description?, tags[],
          createdAt, updatedAt, lastOpenedAt?, metadataStatus
Tag:      id, name, normalizedName, createdAt
Preference: viewMode, sort, selectedTag
```

### UI guidance

Make “Add bookmark” quick enough to use repeatedly. Show title, domain, and tags in a readable list. Support search and tag filtering without making the user navigate to a separate search screen. Provide a clear empty state with one example action. Use a new-tab or same-tab choice only if it is genuinely useful; do not add a settings maze for browser behavior.

### Online boundary

If metadata fetching is added, allow the bookmark to save even when it fails. Show a small pending or unavailable status rather than an error page. Rate-limit or defer remote requests, avoid sending URLs to third parties without explaining it, and never let a remote favicon or title request become the source of truth.

### Tests

Test URL validation, duplicate policy, long titles, tags with spaces or unusual characters, search with no results, import/export, opening links offline, metadata failure, keyboard navigation, and layouts with long domains. Ensure imported URL text is escaped and not injected into the page.

## Small local multiplayer game

### Product decision

A local multiplayer game can be a good PWA when the core play is browser-local, turn-based, pass-and-play, or uses a deliberately limited peer/local transport. A PWA is not automatically appropriate for a game requiring authoritative low-latency online networking, continuous background execution, or native-level performance. State the boundary instead of pretending the app is universally offline multiplayer.

### Default architecture

Keep gameplay logic in a deterministic domain module separate from rendering and input. Store settings and resumable local sessions in IndexedDB or `localStorage` according to their size and structure. Use a compact serializable state snapshot with a game version. Avoid a backend, accounts, or a networked lobby unless the user asks for online play.

```text
GameSettings: sound, theme, difficulty, controlHints
Session: id, gameVersion, players[], stateSnapshot,
         turn, startedAt, updatedAt, status
Result: id, sessionId, winner?, scores, completedAt
```

### UI guidance

Get to play quickly: mode selection, player setup, then the board or playfield. Keep in-game controls large and obvious on a phone. Avoid showing persistence or PWA mechanics during play unless a save, restore, or offline state needs explanation. Provide a clear pause/resume path and an end-of-game summary with a new-game action.

For pass-and-play, design turn handoff intentionally so the next player cannot accidentally see hidden information. For peer connectivity, show connection state, whose turn is authoritative, retry behavior, and what happens if the connection is lost. Do not call a local-only game “online multiplayer” merely because it uses a web socket in development.

### Tests

Test deterministic rules, turn transitions, save/resume, refresh during a turn, malformed session state, version upgrades, viewport orientation, touch controls, keyboard controls, reduced motion, and network loss when a remote mode exists. Verify that a completed game cannot be accidentally overwritten by a stale resume action.

## Personal notes and productivity app

### Product decision

A notes or personal productivity app is an especially good local-first PWA candidate when quick capture, organization, and offline access matter more than multi-device collaboration. Do not add accounts or sync by default. If synchronization later becomes necessary, introduce it as a separately designed capability with clear conflict behavior.

### Default architecture

Use IndexedDB for notes, tasks, labels, and drafts. Keep the editor state separate from persisted records so the UI can provide autosave without coupling every keystroke to storage. Persist a draft or edit session when it protects against reload loss. Store plain text or structured editor data unless rich text is a demonstrated requirement.

```text
Note: id, title, body, labels[], pinned, archived,
      createdAt, updatedAt
Task: id, title, completed, dueOn?, noteId?, createdAt, updatedAt
Draft: id, targetType, targetId?, body, updatedAt
Preference: theme, sort, defaultView
```

### UI guidance

Make capture the shortest path: a prominent new-note or new-task action, a focused editor, and a visible saved state. Put search, labels, pinned items, and archive behind simple navigation rather than a dense productivity dashboard. Preserve the draft if the user navigates away or the browser reloads. Use an undo action for deletion and make archive distinct from delete.

### Data ownership and export

State plainly that notes remain on the device unless the app later gains a documented sync feature. Offer export in a readable format where appropriate in addition to the structured backup format. Import should preview counts and allow the user to merge or replace. Do not silently deduplicate based on title alone; use stable IDs and an explicit collision policy.

### Tests

Test capture, autosave boundaries, draft recovery, search, label changes, archive/restore, deletion undo, export/import round-trips, schema migration, offline reload, keyboard shortcuts where supported, screen-reader naming, and long-note performance. Verify that a storage error never presents an unsaved note as saved.

## How to adapt these examples

When a new request resembles one of these examples, copy the decision pattern rather than the feature list. For example, a recipe collection shares the bookmark manager’s URL/metadata boundary but may need richer structured content; a habit tracker shares the finance tracker’s derived-summary approach but needs careful date and recurrence semantics; a local puzzle game shares the game’s deterministic-state and resume model.

For every adaptation, explicitly answer:

| Decision | Required answer |
| --- | --- |
| Offline value | What must remain useful without a connection? |
| Source of truth | Which local records are authoritative? |
| Storage | Why is each field stored in `localStorage`, IndexedDB, Cache API, or OPFS? |
| Recovery | How does the user undo, export, restore, or migrate? |
| Online boundary | What is optional, what is required, and what fails gracefully? |
| UI restraint | Which screen, setting, or dependency was deliberately omitted? |
