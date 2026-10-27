# Contract: Web App Manifest and Offline Behavior

Defines what the installed app reports about itself and how offline and updates behave (FR-048, FR-049, FR-050).

## Manifest

All URLs are relative to the deployment base path (`/<repo-name>/` on GitHub Pages, set by the `BASE_PATH` build variable).

| Field | Value |
|---|---|
| `name` | "Project Blocks" (provisional, BRD O1) |
| `short_name` | "Blocks" |
| `description` | Short original description |
| `start_url` | `./` (resolves inside the base path) |
| `scope` | `./` |
| `display` | `standalone` |
| `orientation` | `any` (portrait is the primary phone layout) |
| `background_color`, `theme_color` | Match the light theme background; dark theme set via `<meta name="theme-color">` media variants |
| `icons` | Original 192 px and 512 px PNG icons, plus a maskable 512 px icon |

Rules:

- No field may contain the genre trademark or a confusingly similar name.
- Icons are original artwork recorded in `ASSETS.md`.

## Offline behavior

- The build is precached completely during the first visit. After that, every screen and mode works with no network.
- The app makes no network requests during play other than loading its own files. No analytics, fonts or third-party calls.
- Caches are namespaced with the app id; the service worker deletes only caches it created.

## Updates

- A new version downloads in the background and **waits**. It takes over when the app is next launched (all old windows closed), so a game in progress is never reloaded.
- Saved data is untouched by updates ([storage-schema.md](storage-schema.md)).
- The service worker scope equals the base path, so it cannot control other project sites on the same origin.
