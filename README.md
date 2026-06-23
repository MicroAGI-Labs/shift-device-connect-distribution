# Shift Device Connect — Distribution Page

Landing page for sharing Shift Device Connect app download links (TestFlight + APK).

**Live:** https://microagi-labs.github.io/shift-device-connect-distribution/

## Setup

1. Enable GitHub Pages: **Settings > Pages > Deploy from branch > main / root**
2. Create a release with the APK file named `shift-device-connect.apk`

## Updating the APK

1. Go to **Releases** > **Create a new release**
2. **Tag:** semantic version, e.g. `v1.0.0`, `v1.1.0`, `v2.0.0`
3. **Title:** date, e.g. `2026-03-10`
4. **Description:** changelog (what's new, fixes, etc.)
5. Attach the APK file — filename must be **`shift-device-connect.apk`**
6. Click **Publish release**

The download link on the site always points to the latest release's `shift-device-connect.apk`.

## Updating `version.json` (required for in-app update banner)

`version.json` is the source of truth the **MicroAGI backend** polls to decide whether the operator app should show an "App update available" banner. It must be updated **every time** a new APK / TestFlight build is published.

After publishing the release:

1. Edit `version.json` on `main`.
2. Set the fields:
   - `version` — semver string (e.g. `"1.0.1"`) **without a `v` prefix**. Must match `app.json`'s `version` in the operator repo for the build you're shipping.
   - `iosBuildNumber` — the iOS production build number from EAS (the integer EAS auto-increments via `appVersionSource: "remote"`).
   - `androidVersionCode` — the Android build's `versionCode`.
   - `androidDownloadUrl` / `iosDownloadUrl` — usually unchanged.
   - `publishedAt` — ISO date (`YYYY-MM-DD`).
   - `releaseNotes` — short summary; surfaces in the update banner.
3. Commit and push to `main`. GitHub Pages picks it up within a couple of minutes; the backend's `ShiftAppReleaseService` cache TTL controls how quickly clients see the bump after that.

The operator app compares its local `Constants.expoConfig.version` (from `app.json`) against `version.json`'s `version` via a semver compare. Mismatched prefixes (`v1.0.1` vs `1.0.0`) silently break the check — keep the field free of `v` prefixes here, even if the GitHub release tag uses one.
