# Recent PWA decisions

These decisions reflect the current personal-PWA architecture used in this skill.

- **Local-first is the default.** IndexedDB is the primary storage for user-created structured data; `localStorage` is for tiny preferences and legacy migration only.
- **Dexie.js is a practical default wrapper** for IndexedDB in small apps when it materially improves schema/migration/transaction ergonomics. It is not mandatory.
- **Browser persistence is not backup.** `navigator.storage.persist()` may reduce automatic eviction but cannot protect against deliberate clearing of site/app data.
- **Recovery is file-based by default.** For high-value local data, provide an export/import path that produces a user-visible file outside browser-managed origin storage.
- **Cloud backup is optional.** If there is no multi-device sync requirement, do not introduce a server database solely for backup. Add cloud providers only as optional backup destinations when the user wants off-device automation.
- **Encryption is client-side when privacy matters.** Encrypt the backup before it leaves the device. Do not rely only on provider-side encryption when the requirement is provider-unreadable backups.
- **Backup reminders should be non-intrusive.** Prefer “last backup is old” or “meaningful changes since backup” rather than a mandatory daily prompt.
- **Never expose developer prompts in the UI.** Implementation instructions are not user-facing product copy; generation prompts must not be hardcoded into rendered components.
- **Do not over-normalize during an initial refactor.** A first migration can preserve an existing whole-object persistence pattern for safety; optimize into finer-grained records later when the schema and UI are stable.
