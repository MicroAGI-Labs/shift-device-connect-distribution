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
