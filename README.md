# Shift Device Connect — Distribution Page

Landing page for sharing Shift Device Connect app download links (TestFlight + APK).

**Live:** https://microagi-labs.github.io/shift-device-connect-distribution/

## Setup

1. Enable GitHub Pages: **Settings > Pages > Deploy from branch > main / root**
2. Create a release with the APK file named `shift-device-connect.apk`

## Publishing a new release

Every release uploads **two files**: the APK and a `version.json`. The MicroAGI backend reads `version.json` to decide whether to show an "App update available" banner in the operator app.

### Step 1 — Build

Run an EAS production build (Android `preview` profile for the APK, iOS `production` profile for TestFlight) in the operator repo. Note both:

- iOS `Build number` from EAS (auto-incremented integer, e.g. 23)
- Android `Version code` from EAS (auto-incremented integer, e.g. 4)

### Step 2 — Compose `version.json`

Copy `version.json` from this repo's root (the file at `main` is a reference template — its values are the *previously released* version's values) and edit the fields for the new release:

```json
{
  "version": "1.0.1",
  "iosBuildNumber": 24,
  "androidVersionCode": 5,
  "androidDownloadUrl": "https://github.com/MicroAGI-Labs/shift-device-connect-distribution/releases/latest/download/shift-device-connect.apk",
  "iosDownloadUrl": "https://testflight.apple.com/join/z2VEmVXp",
  "publishedAt": "2026-06-23",
  "releaseNotes": "Short summary of changes; surfaces in the in-app banner."
}
```

| Field | Notes |
|---|---|
| `version` | Semver, **no `v` prefix**. Must match the `version` field in the operator repo's `app.json` for the build you're shipping. |
| `iosBuildNumber` | Integer from EAS. Optional for the banner check, but useful to record. |
| `androidVersionCode` | Integer from EAS. Same. |
| `androidDownloadUrl` / `iosDownloadUrl` | Usually unchanged release-to-release. |
| `publishedAt` | ISO date (`YYYY-MM-DD`). |
| `releaseNotes` | Short summary; surfaces in the banner. |

### Step 3 — Create the GitHub release

1. Go to **Releases > Draft a new release**.
2. **Tag:** e.g. `v1.0.1`. (The tag string itself doesn't matter to the backend — it reads `version.json`, not the tag — but keep `v`-prefixed tags for GitHub convention.)
3. **Title:** date, or `v1.0.1`, whichever you prefer.
4. **Description:** human-readable changelog.
5. **Attach both files:**
   - `shift-device-connect.apk`
   - `version.json` (the one you composed in Step 2)
6. Click **Publish release**.

### Step 4 — (Optional) Update `version.json` on `main`

Commit the same `version.json` to `main` so anyone browsing the repo at a glance sees the current shipped version. Not required for the banner — the backend reads only the release asset — but it keeps the repo honest and gives the next release a fresh template.

### How the backend consumes it

The backend's `ShiftAppReleaseService` fetches `https://github.com/MicroAGI-Labs/shift-device-connect-distribution/releases/latest/download/version.json` and caches the result for **5 minutes**. Once a new release is published, app clients see the new version on the next foreground after the cache window expires.

The operator app compares its local `Constants.expoConfig.version` (from `app.json`, baked at build time) against `version.json.version` via a semver compare. The backend defensively strips a leading `v` if present, but the convention in `version.json` is no prefix.
