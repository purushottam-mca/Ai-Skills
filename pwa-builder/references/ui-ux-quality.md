# UI/UX Quality Reference

Use this reference when designing or reviewing the interface of a personal PWA. Optimize for a calm, obvious, mobile-first experience that remains capable on desktop. Do not turn the review into a generic visual-design exercise; judge whether the product makes its main task easy.

## Contents

1. [Experience model](#experience-model)
2. [Mobile-first layout](#mobile-first-layout)
3. [Navigation](#navigation)
4. [Forms and input](#forms-and-input)
5. [State design](#state-design)
6. [Accessibility review](#accessibility-review)
7. [Desktop and responsive review](#desktop-and-responsive-review)
8. [Visual quality review](#visual-quality-review)
9. [Final walkthrough](#final-walkthrough)

## Experience model

Describe the app in one sentence before selecting screens:

> “A person uses this app to **[primary action]** on **[their data]**, usually from **[context/device]**, and should reach a useful result in **[short flow]**.”

Use that sentence to reject decorative or secondary UI that competes with the main action. Define one primary action for each screen or mode. If the interface needs several equal primary actions, revisit the information architecture.

Prefer direct manipulation and local feedback. When the user adds a record, show it immediately from local state and persist it without making the user wait for a remote response. When a value is derived, explain the source succinctly rather than forcing the user to navigate to a separate report.

## Mobile-first layout

Start at a narrow phone viewport and add space progressively. Keep the most frequent action within comfortable thumb reach when the flow permits. Use bottom actions or a sticky action region only when it does not cover content or the keyboard. Account for safe areas on devices with rounded corners or home indicators.

Use a restrained layout system. The default mobile shell should feel like a real mobile app rather than a responsive desktop site: use a bottom tab bar when the app has a small number of peer-level destinations, keep labels visible, show a clear active state, respect safe-area insets, and keep the bar stable while navigating. Do not force bottom tabs onto a single-task app or a flow where tabs would compete with the task.


| Element | Preferred behavior |
| --- | --- |
| Page title | State the current context; avoid marketing language. |
| Primary action | One obvious action with a clear verb. |
| Secondary actions | Group behind a compact menu, toolbar, or contextual action. |
| Lists | Show useful summary information; avoid making every row a mini-dashboard. |
| Detail view | Prioritize reading and editing; keep destructive actions separated. |
| Dialog | Use only for focused tasks or consequential decisions. |
| Navigation | Keep destinations few and peer-level; use labeled bottom tabs on mobile when that pattern fits. |

Do not use a dense desktop table as the default mobile layout. Convert records into readable cards, grouped rows, or a focused detail screen unless column comparison is the actual job. Preserve the user’s ability to scan and act without horizontal scrolling.

Use touch-friendly controls with visible pressed and focus states. Avoid tiny icon-only buttons for important actions; if an icon is necessary, provide an accessible name and a nearby or discoverable label. Do not make swipe, drag, hover, or long-press the only way to complete a task.

## Navigation

Choose the simplest pattern that matches the number of destinations:

| Product shape | Default navigation |
| --- | --- |
| One primary task | Single focused screen with inline panels or a detail route |
| A few peer destinations | **Default:** labeled bottom tabs on mobile, compact top/side navigation on desktop |
| List plus detail | List → detail → edit, with predictable back navigation |
| Settings and export | One overflow or settings entry; avoid a separate settings dashboard |
| Game modes | Start screen → mode/session → results; keep in-game controls focused |

Maintain location awareness. The title, selected tab, active tab styling, back affordance, and URL/route should agree. Keep the bottom tab bar available for peer destinations but hide it or replace it with focused back navigation inside modal-like creation/edit flows when it would distract from completion. Preserve list filters and scroll position when returning from detail where practical. Avoid nested drawers and navigation stacks that make local tools feel like enterprise software.

Use URLs when they materially improve reload, sharing, browser history, or direct access. Do not add a router solely because a framework makes one available. If the app is truly single-view, a small view-state model may be clearer.

## Theme and appearance

For repeated-use personal apps, include **System**, **Light**, and **Dark** appearance choices unless the product has a strong reason not to. Apply the selected theme before the first meaningful paint, persist it locally, and use shared design tokens so the bottom tab bar, active states, forms, dialogs, charts, and empty/error states remain legible in every theme. Test switching themes while a tab, form, or dialog is active; the user should not lose navigation state or unsaved input.

## Forms and input

Design forms around decisions rather than database fields. Ask only for information needed at that point. Use labels that remain visible, examples only when ambiguity exists, and input types that bring up the right mobile keyboard.

Keep validation close to the field and summarize unresolved problems near the submit action. Preserve all valid entered data when a field fails. Do not clear a form after a validation error or after a transient storage/network failure. Use `aria-describedby` or an equivalent accessible relationship for help and error text.

Use an explicit save action when edits are consequential or multi-field. Use autosave for low-risk drafts only when the user can see the save status and undo or restore behavior is clear. Do not mix autosave and manual save in a way that makes the source of truth ambiguous.

For numeric and financial input, normalize carefully without changing the user’s displayed value unexpectedly. Distinguish zero, blank, invalid, and unavailable. For dates, distinguish a date-only value from a timestamp and explain timezone behavior when it affects the result.

## State design

Every important screen should have intentional states. Build them before polishing the happy path.

| State | Good behavior |
| --- | --- |
| First-use empty | Explain the value and offer one clear first action. |
| Empty after filtering | Explain that the filter, not the app, produced no results; offer to clear it. |
| Loading | Keep the layout stable and show progress only where waiting is real. |
| Saving | Give local confirmation promptly; expose pending sync separately if needed. |
| Offline | Let local work continue; explain only the capabilities affected. |
| Network failure | Preserve local data, state what failed, and offer retry without duplicating the action. |
| Storage failure | Avoid pretending data was saved; offer export/recovery or a retry path. |
| Validation error | Point to the field, explain the correction, and preserve other input. |
| Destructive action | Use undo or a confirmation only when recovery is not obvious. |
| Updated version | Tell the user a new version is ready and let them choose when to reload. |

Use status text, toasts, inline notices, or banners according to consequence and duration. A toast should not be the only record of a critical error. Do not let an offline banner permanently consume the main viewport when it adds no decision value.

## Accessibility review

Perform a keyboard pass without a mouse or touch device. Check that every interactive control is reachable in a logical order, focus is visible, focus is not trapped accidentally, dialogs return focus appropriately, and no action depends on hover or pointer movement.

Perform a semantic pass. Use buttons for actions, links for navigation, headings in logical order, lists for lists, labels for controls, and native form semantics before adding ARIA. Give icon-only controls accessible names. Announce meaningful asynchronous state changes without flooding the user with updates.

Perform a visual pass. Check contrast, text resizing, zoom, focus visibility, reduced motion, dark mode if supported, and meaning that is conveyed without color. Do not rely on placeholder text as the label. Make error states understandable to users with color-vision differences.

Perform an interaction pass with a screen reader or accessibility tree inspection where available. Verify that the app’s purpose, current location, control names, validation messages, list counts, and save/offline status are intelligible. Test a real device when the app relies on touch, viewport behavior, or installation UI.

## Desktop and responsive review

At desktop widths, use space to reduce scanning effort. A two-column layout may pair a list with detail; a max-width content region may improve reading; a side panel may make filters visible. Do not stretch short forms across the entire screen or fill space with decorative cards.

Support mouse, keyboard, and trackpad input without creating separate mental models. Allow common keyboard actions when they are discoverable and do not conflict with browser behavior. Keep hover enhancements supplemental; all core information and actions must remain available without hover.

Review at least four layout bands: narrow portrait phone, wide portrait or landscape phone, tablet, and desktop. Look for clipped dialogs, keyboard overlap, fixed elements covering content, broken focus order, long labels wrapping poorly, and controls that become too far apart to operate efficiently.

## Visual quality review

Use a small design vocabulary: a restrained color system, a clear type scale, consistent spacing, a modest radius system, and a limited elevation treatment. Favor contrast and hierarchy over decoration. Use color to distinguish meaning, not to create noise.

Check whether the interface feels trustworthy for data the user owns. Export/import, storage, sync, and delete actions should be findable without turning the first screen into a settings manual. Explain local-only behavior briefly when it reduces uncertainty.

Avoid common AI-generated UI symptoms: excessive gradients, repeated rounded cards, decorative statistics with no task value, oversized hero sections, vague labels such as “Continue,” hidden primary actions, too many toggles, modal-heavy flows, arbitrary emoji icons, and a generic dashboard imposed on a simple utility.

## Final walkthrough

Walk through the app as a first-time mobile user, a returning offline user, and a desktop keyboard user. For each persona, record the first meaningful action, the point of confusion, the recovery path after a failure, and whether the app preserves trust.

Finish by asking:

1. Can the purpose be understood immediately?
2. Can the primary task be completed with one hand on a phone?
3. Can the user tell what is saved locally and what requires the network?
4. Can the user recover from accidental deletion, bad input, failed storage, and failed network activity?
5. Can a keyboard and assistive-technology user complete the same task?
6. Does the desktop layout improve efficiency rather than merely enlarge the mobile layout?
7. Does the mobile shell use bottom tabs only where they clarify peer destinations, rather than as decoration?
8. Do System, Light, and Dark modes preserve readability and state visibility?
9. Which screen, dialog, dependency, or setting could be removed without reducing value?

Remove the item identified in the final question unless there is a concrete reason to keep it.
