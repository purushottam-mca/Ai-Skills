---
name: pwa-builder
description: Build personal and small Progressive Web Apps as polished, mobile-app-like, offline-first, local-first products. Use when creating, extending, reviewing, refactoring, or deploying a small PWA, especially utilities, productivity tools, trackers, finance tools, notes apps, bookmarks, experiments, or local games.
---

# Personal PWA Builder

Build small PWAs as **lightweight, private, local-first products**, not as miniature enterprise systems. Apply the following rule throughout the task:

> **Do not add complexity merely because it is technically possible.**

Treat this skill as an architecture-and-product-design decision framework. It must influence both implementation choices and UI/UX decisions. The default product feel is a focused mobile app: bottom tab navigation for a small set of peer destinations, a compact top bar, touch-friendly controls, and light/dark appearance settings. Keep the core principles framework-agnostic; use vanilla JavaScript, React, Vue, Svelte, or another suitable stack only when it improves the particular app.

## Operating workflow

For every new PWA request, reason through these stages in order and record the decision briefly:

1. **Requirement:** Identify the smallest useful feature set, the primary user, the important data, and the actions the user performs most often.
2. **PWA fit:** Decide whether an installable, local-first web app is appropriate. If not, explain why and recommend a simpler or more suitable alternative.
3. **Offline requirements:** Classify each capability as offline-required, offline-preferred, or online-required. Do not promise offline behavior for a feature that depends on a remote service.
4. **Data model:** Define entities, identifiers, timestamps, relationships, validation, deletion behavior, and migration needs before choosing storage.
5. **Storage:** Select the smallest storage mechanism that satisfies the data shape, volume, query, durability, and file requirements.
6. **Navigation:** Design the fewest screens and navigation actions that make the main tasks obvious. Default to bottom tab navigation for roughly three to five genuinely peer-level mobile destinations; use a simpler single-screen or list/detail flow when tabs would add clutter.
7. **UI/UX:** Design mobile-first, one-handed flows with clear hierarchy, useful empty states, accessible controls, and restrained visual polish.
8. **PWA capabilities:** Add a valid manifest, service worker, icons, installability support, update handling, and offline shell behavior appropriate to the app.
9. **Security and privacy:** Keep data under user control. Minimize collection, permissions, third-party services, secrets, and network exposure.
10. **Testing:** Test core tasks, reloads, offline use, installation, storage durability, migrations, accessibility, responsive layouts, and failure states.
11. **Deployment:** Prefer a static deployment or simple self-hosting. Introduce a backend only when a concrete requirement makes local-only behavior insufficient.

Do not begin by selecting a framework, database, API, authentication provider, or cloud platform. Begin with the requirements and the offline/data decisions. For a personal PWA where the primary risk is accidental loss of local data rather than multi-device synchronization, prefer a file-based backup design over a server database.

## Core principles versus optional choices

### Core PWA principles — enforce these

Prefer offline-first behavior, local-first data ownership, minimal dependencies, fast startup, a polished mobile-app-like shell, bottom-tab navigation when appropriate, responsive desktop support, graceful online/offline transitions, accessible interfaces, installability, light/dark theme support, export/import for valuable data, and simple static deployment.

Keep the app small in scope. Prefer obvious actions, shallow navigation, direct manipulation, whitespace, visual hierarchy, and a small number of purposeful dialogs. Avoid spreadsheet-like layouts unless dense tabular editing is genuinely the user’s central task. For most persistent personal PWAs, provide **System**, **Light**, and **Dark** appearance choices, persist the selection locally, and apply it before the first paint to avoid a distracting flash. Do not let theme polish delay the core task, but do not omit it when the app is intended for repeated use.

### Optional implementation choices — justify each one

Frameworks, routing libraries, state-management libraries, UI kits, IndexedDB wrappers, client-side databases, background synchronization, push notifications, cloud storage, APIs, Docker, and authentication are implementation options, not requirements. Select them only after demonstrating the problem they solve and the cost they introduce.

If a simple IndexedDB-backed static app solves the request, do not introduce PostgreSQL, Redis, authentication, REST, GraphQL, cloud storage, microservices, Docker, or a complicated state-management framework merely because they are familiar or available.

## Decide whether PWA is the right choice

Choose a PWA when the app is primarily a web UI, benefits from installation or a home-screen entry point, should work with intermittent connectivity, stores modest user-controlled data, and can use browser capabilities without privileged native APIs.

Do not force a PWA when the main requirement is a high-performance 3D or compute-heavy application, continuous background execution, deep operating-system integration, reliable access to specialized hardware, large local files beyond practical browser limits, regulated centralized records, multi-user real-time collaboration, or guaranteed cross-device synchronization. For those cases, explain the limitation and consider a native app, a server-backed web app, or a hybrid approach.

A PWA may still be a good offline client for a larger system, but do not disguise a synchronization-heavy product as a local-only app. State clearly which data is authoritative and how conflicts are resolved.

## Default architecture for a small offline-first PWA

Use this as the default unless the requirements disprove it:

| Layer | Default choice | Add complexity only when |
| --- | --- | --- |
| UI | Semantic HTML, CSS, and a small component layer with a mobile-app-like shell | Reuse, routing, or state complexity makes a framework materially clearer |
| Application state | In-memory view state plus a small domain/service layer | Multiple complex views need coordinated derived state |
| Persistent structured data | IndexedDB through a thin repository or Dexie.js when it materially reduces IndexedDB boilerplate | A different local database provides a demonstrated advantage |
| Preferences | `localStorage` for small non-critical key/value settings | Data is structured, queryable, large, or must be transactionally updated |
| Static assets | Bundled files and the Cache API through a service worker | Runtime caching is required for a specific remote asset class |
| Large binary files | OPFS or another browser file mechanism when appropriate | Files must be synchronized, shared, or exceed practical local limits |
| Network | No backend by default | Remote data, collaboration, sync, payments, notifications, or protected computation is required |
| Deployment | Static hosting over HTTPS or simple self-hosting | Server-side behavior is a concrete requirement |

Keep domain logic independent from UI components and storage APIs. A useful small structure is:

```text
src/
  app/                 app shell, routing, composition
  components/          reusable presentational components
  features/            feature-specific screens and domain actions
  domain/              entities, validation, business rules
  data/                repositories, schema version, migrations, import/export
  pwa/                 manifest helpers, service-worker registration, update UI
  styles/              tokens, global styles, responsive rules
  tests/               unit, integration, accessibility, and browser tests
public/
  icons/               install icons and maskable icon where needed
  manifest.webmanifest
  offline.html         optional fallback for navigation requests
```

Adapt the structure to the chosen stack. Do not create empty abstractions or directories for hypothetical features.

## Storage decision rules

Choose storage based on data shape and behavior rather than trend. Keep storage behind a repository so UI code does not directly depend on IndexedDB calls.

| Mechanism | Use for | Do not use for |
| --- | --- | --- |
| `localStorage` | Tiny preferences such as theme, sort order, dismissed hints, or last-selected view | Collections, large records, secrets, frequent writes, or data that needs transactions |
| IndexedDB | Persistent structured records, collections, indexes, drafts, queues, and offline app data | A trivial single preference that needs no querying |
| Cache API | Versioned static assets and deliberately cached HTTP responses | Primary business data or records requiring domain queries |
| Service worker | Intercepting requests, serving the app shell, managing cache versions, and coordinating offline behavior | Acting as the source of truth for application data |
| OPFS | Large local files, binary workspaces, media, or high-volume file operations where browser support and product needs justify it | Ordinary notes, settings, or small records |

Use IndexedDB for structured user data by default. For small PWAs, Dexie.js is a reasonable default wrapper when it improves schema versioning, transactions, and maintainability without becoming a second application framework. Store stable IDs, explicit schema versions, `createdAt` and `updatedAt` timestamps where useful, and a predictable deletion policy. Keep derived values recomputable when practical instead of persisting every calculation.

## Persistence, migrations, and data safety

Define a schema version from the first release. On opening storage, migrate deterministically from each supported prior version to the current version. Make migrations idempotent where feasible, preserve unknown fields when safe, and never silently discard user data. For high-risk migrations, preserve the previous source until the migrated records have been read back and validated. Mark migration complete only after the post-migration integrity check succeeds. If migration fails, keep the original data recoverable, show a clear recovery state, and do not continue as if the database were empty.

Debounce non-critical writes but save user-entered data promptly enough to prevent surprise loss. Avoid writing on every keystroke when it creates unnecessary churn; use draft state and explicit or short-idle persistence. Confirm destructive actions when recovery is not obvious. Prefer soft deletion or an undo window for valuable records.

Treat import/export as a default product capability for any PWA containing user-created records that would be costly to lose. Provide visible **Export data** and **Import data** actions, ideally in Settings or the app menu and not buried in developer-only controls. Use a documented, versioned JSON format for structured records and include metadata such as schema version, export timestamp, and app identifier. Validate imports before changing existing data, show a preview or summary, reject malformed or incompatible files safely, and offer merge versus replace semantics when both are meaningful. Never overwrite the current dataset without an explicit user action and a backup opportunity. A browser-only copy is not a disaster-recovery backup: users can explicitly clear site/app data, which can remove origin storage. For high-value data, provide a user-visible file backup that can live outside browser-managed storage. Do not promise that `navigator.storage.persist()` prevents deliberate deletion.

## Backup and recovery strategy

Treat **local storage as the source of truth, not as the only copy** when losing the data would be costly. Browser persistence APIs reduce accidental eviction risk but cannot protect against explicit user deletion, browser/profile removal, uninstall flows, or other origin-data clearing actions. The recovery design must therefore distinguish:

```text
Browser-managed working copy
  IndexedDB / Dexie
        │
        └── Export / Backup
              │
              ▼
        User-visible file
        (.json or encrypted .enc)
              │
              └── optional user-chosen cloud / USB / NAS / filesystem
```

For a personal PWA without sync requirements, prefer **local-first + user-controlled backup** over introducing a backend merely for disaster recovery. Add cloud storage only when the user wants automatic off-device backup and the provider/integration is justified. Keep the backup provider pluggable rather than coupling the core data model to a cloud vendor.

Use `navigator.storage.persist()` opportunistically to reduce browser eviction under storage pressure. Check whether persistence was actually granted. Do not request it in a way that creates an unexpected permission prompt on first paint; request it when the app has meaningful user data or at an appropriate onboarding/settings moment. Persistence is not a backup and does not protect against explicit user deletion.

If the app offers backup reminders, prefer a **staleness-based or change-based reminder** (for example, after several days without a successful backup or after meaningful new data) rather than an intrusive daily prompt. Always show the last successful backup time or status somewhere discoverable.

If encrypted backups are implemented, encrypt **before** writing/uploading the backup and keep the key material out of the backup. Prefer authenticated encryption such as AES-GCM through Web Crypto and a vetted password-based key-derivation implementation. Clearly communicate that losing the encryption password/key can make the backup unrecoverable. Do not invent cryptography.

When a user selects a real filesystem location through a browser file/directory picker, treat the selected handle as user-authorized external storage. Do not assume a PWA can silently write arbitrary files to the device filesystem on every browser/device.

## Offline-first and service-worker strategy

Design the app shell to load without a network after it has been installed or visited successfully. Treat local data operations as the normal path, not as an exception branch. When online functionality exists, isolate it behind an explicit boundary and define what the app does when it is unavailable.

Use a versioned precache for the minimal shell and immutable build assets. Use a clear cache naming scheme and delete obsolete caches during activation. Prefer cache-first for versioned static assets, network-first with a bounded fallback for content that must be fresh, and stale-while-revalidate only when displaying slightly stale content is acceptable. Do not cache private responses or authenticated data unless the security model explicitly supports it.

Handle service-worker updates deliberately. Notify the user when a new version is ready, avoid interrupting an in-progress edit, and provide a predictable reload/update action. Do not create a reload loop. Test first install, repeat load, update, old-cache cleanup, offline navigation, and a failed asset request.

Use an explicit online/offline indicator only when it helps the user make a decision. The browser’s connectivity event is a hint, not proof that a server is reachable. For online operations, show pending, retry, success, and failure states based on actual requests. Never block local reading or editing merely because the network is unavailable.

## PWA manifest and installation requirements

Include a valid web app manifest with a human-readable name, short name, start URL, display mode appropriate to the app, theme and background colors, and suitable icons. Ensure the app has a meaningful title, mobile viewport configuration, HTTPS in deployment, and a registered service worker. Use maskable icons when the design supports them, and verify that icons remain legible at small sizes.

Make installation useful rather than ornamental. The installed experience should open to the expected route, preserve local data, provide sensible back navigation, and avoid presenting browser-only assumptions. Do not aggressively prompt for installation; explain the value at an appropriate moment or let the platform provide its normal affordance.

## Mobile-first and responsive UI rules

Design the narrow mobile layout first. Make the default shell feel like a well-designed mobile app rather than a shrunken desktop website. Use bottom tabs as the primary navigation pattern when the app has a small number of peer-level sections; each tab must have a clear label, a stable destination, an active state, and an accessible name. Keep the tab bar clear of transient actions, and move a primary create/add action above the bar or into a focused screen. Put the primary action within comfortable thumb reach when practical, use large enough touch targets, avoid hover-only meaning, keep important controls visible, and prevent accidental destructive taps. Use progressive disclosure for secondary options rather than adding permanent toolbar clutter.

Make the main task clear within seconds. Use one obvious primary action per context, concise labels, strong hierarchy, restrained decoration, and consistent spacing. Prefer a small number of meaningful screens over a dashboard full of cards. Use bottom tabs by default for a small set of peer-level mobile destinations; otherwise use a simple top bar, back navigation, tabs, or a single focused screen. On desktop, adapt the same destinations into a compact top or side navigation without changing the information architecture.

### Theme and appearance

Provide **System**, **Light**, and **Dark** choices for repeated-use personal apps unless there is a strong product reason not to. Store the preference locally, honor the system preference when System is selected, use design tokens rather than scattered hard-coded colors, and ensure controls, charts, dialogs, borders, and empty/error states remain legible in both themes. Test theme changes while a form, dialog, or bottom tab is active; do not reload the app just to switch themes.

On larger screens, use the extra space to improve reading, grouping, and keyboard efficiency rather than merely enlarging mobile controls. Constrain line length, avoid excessive empty chrome, support pointer and keyboard input, and use responsive layouts that preserve task order. Test portrait phone, landscape phone, tablet, and desktop widths.

For forms, group fields by user intent, use appropriate input types and autocomplete, preserve entered values on validation errors, label every control, provide inline guidance near the point of need, and avoid multi-step forms for small tasks. Use sensible defaults without hiding important choices. Support submit with the keyboard and make validation understandable without relying only on color.

Implement meaningful empty, loading, error, and confirmation states. Empty states should explain what the app does and provide the next action. Loading states should be brief and localized. Errors should explain what happened, what remains safe, and how to recover. Confirm only consequential actions; do not add confirmation dialogs for reversible or routine interactions.

## Accessibility, performance, privacy, and dependencies

Use semantic elements, visible focus styles, logical heading order, keyboard access, accessible names, sufficient contrast, reduced-motion support, and status announcements for important asynchronous changes. Ensure dialogs trap focus correctly and return focus when closed. Do not encode essential meaning with color, hover, animation, or iconography alone.

Keep the initial experience fast. Ship only necessary JavaScript and assets, lazy-load secondary routes or expensive features, avoid oversized images, minimize layout shift, virtualize only when a real scale problem exists, and keep storage/network work off the critical interaction path. Measure before optimizing, but treat slow startup and janky primary interactions as defects.

Collect and transmit as little data as possible. Do not add analytics, third-party fonts, trackers, remote logging, cloud sync, or permissions unless the user’s requirement justifies them and the privacy impact is clear. Never store credentials or sensitive secrets in browser storage. Treat imported files as untrusted input, validate content, escape rendered text, and protect any backend endpoints if a backend is introduced.

Prefer platform APIs and a small number of well-maintained dependencies. Add a dependency only when it removes meaningful complexity, improves reliability, or provides a capability that would be unreasonable to implement locally. Record why each non-obvious dependency exists and remove unused packages. Do not add a global state framework, database client, router, component library, or utility library by reflex.

## When to introduce a backend or authentication

Introduce a backend only for a concrete requirement such as shared data across devices, multi-user collaboration, server-side secrets, protected computation, remote integrations, centralized records, payments, push delivery, or a dataset too large or dynamic for local ownership. When adding one, preserve local usability where practical and document the source of truth, synchronization model, conflict behavior, failure behavior, and data retention.

Authentication is justified only when the app must identify a user to protect or synchronize data, enforce permissions, meet a compliance requirement, attribute shared actions, or connect to a user-specific external service. Do not add accounts to a personal offline utility merely to make it feel like a production SaaS product. Explain the added onboarding, recovery, privacy, and availability costs.

## Testing and deployment

Test the highest-value user journeys before polishing secondary features. Cover a clean first run, normal CRUD operations, refresh and browser restart, offline reload, local data persistence, export/import, malformed import, migration from an older schema, destructive-action recovery, service-worker update, and online request failure. Add keyboard and screen-reader checks, responsive viewport checks, and at least one real-device install test when possible.

Use unit tests for domain rules and migrations, integration tests for repositories and import/export, browser tests for critical flows and offline transitions, and Lighthouse or equivalent audits as a diagnostic rather than a substitute for task testing. Test with realistic slow-network and no-network conditions. Do not claim offline support until the app has been tested with the network disabled.

Deploy as a static application over HTTPS by default. Make cache invalidation and service-worker updates part of the release process. Keep deployment instructions short and reproducible, document any required headers, and make self-hosting feasible without a proprietary runtime. If a backend is necessary, keep the frontend deployable independently and provide a local development path that does not require production credentials.

## Completion gates

Before calling a PWA complete, run the two checklists below and fix failures rather than merely listing them.

### PWA decision checklist

| Check | Pass condition |
| --- | --- |
| Product fit | The app benefits from web reach, installation, or offline/local behavior, and no better platform is required |
| Scope | The smallest useful feature set is explicit; hypothetical features are not architected in advance |
| Offline contract | Offline-required, offline-preferred, and online-required capabilities are documented |
| Data ownership | The source of truth, persistence behavior, deletion behavior, backup boundary, and recovery path are clear |
| Storage fit | Each storage mechanism is justified by data shape and behavior |
| Complexity budget | No backend, auth, cloud service, API, or major dependency exists without a concrete requirement |
| PWA shell | Manifest, icons, HTTPS deployment, service worker, and expected install behavior work |
| Failure behavior | Offline, reload, update, storage error, import error, and network failure states are usable |
| Privacy | Data collection and third-party requests are minimal and explained |
| Validation | Critical tasks, accessibility, responsive layouts, persistence, and offline behavior are tested |

### UI/UX quality checklist

| Check | Pass condition |
| --- | --- |
| First impression | The purpose and primary action are obvious without configuration |
| Mobile use | The core flow works one-handedly with comfortable touch targets |
| Navigation | There are few destinations, shallow paths, predictable back behavior, and bottom tabs are used when peer destinations justify them |
| Visual hierarchy | Typography, spacing, grouping, and contrast make priorities clear |
| Input | Forms use clear labels, appropriate controls, preserved values, and actionable validation |
| States | Empty, loading, success, offline, error, and destructive-action states are intentional |
| Accessibility | Keyboard, focus, semantics, contrast, reduced motion, and non-color cues work |
| Desktop | Larger layouts improve scanning and efficiency without losing mobile clarity |
| Trust | The app explains local storage, export/import, sync limitations, and destructive actions |
| Theme | System, Light, and Dark choices work consistently when the app is intended for repeated use |
| Backup | User-created records can be exported and safely imported, with validation and collision behavior; high-value data has an external-file recovery path |
| Restraint | No unnecessary dashboard panels, dialogs, animations, dependencies, or settings remain |

## Reference navigation

Read only the reference that matches the current need. Keep this core file as the default operating procedure. The references are deliberately narrow: do not load or implement their advanced patterns unless the request needs them.

| Need | Read |
| --- | --- |
| Storage, schema, migrations, backup, service workers, or deployment detail | [architecture-and-storage.md](references/architecture-and-storage.md) |
| Mobile/desktop interaction, accessibility, states, and visual review | [ui-ux-quality.md](references/ui-ux-quality.md) |
| Concrete guidance for common personal apps | [examples.md](references/examples.md) |

## Common mistakes to avoid

Avoid starting with a SaaS architecture, treating the service worker as a database, storing collections in `localStorage`, confusing cache storage with user-data durability, caching private data casually, assuming `navigator.storage.persist()` is a backup or prevents deliberate deletion, assuming the `online` event proves reachability, losing drafts on reload, silently replacing local data during import, using timestamps as the only record identity, omitting export/import for valuable local records, making a browser-only copy the sole recovery plan for high-value data, omitting a theme choice for a repeated-use app, making installation the primary goal, adding account creation before value is delivered, hiding actions behind unfamiliar gestures, replacing clear bottom tabs with a hamburger menu, using color as the only validation signal, and adding a framework or dependency without a task-level reason.

When requirements are ambiguous, choose the simpler local-first interpretation and state the assumption. Ask a focused question only when the ambiguity changes data ownership, privacy, synchronization, or irreversible behavior.
