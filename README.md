# Shift Device Connect — Distribution Page

Landing page for sharing Shift Device Connect app download links (TestFlight + APK).

**Live:** https://microagi-labs.github.io/shift-device-connect-distribution/

## Setup

1. Enable GitHub Pages: **Settings > Pages > Deploy from branch > main / root**
2. Create a release with the APK file named `shift-device-connect.apk`

## Publishing a new release

The MicroAGI backend manages most of this for you. When you trigger a production build with EAS, the backend gets notified, finds (or creates) an open **draft** release here, and writes the APK + a `version.json` onto it. You just review and publish.

### Per-release flow

1. **`eas build --platform android --profile production`** (operator repo)
2. **`eas build --platform ios --profile production`** (operator repo)
3. **`eas submit --platform ios --latest`** — uploads the iOS build to TestFlight
4. Wait for Apple to finish TestFlight processing (you'll get an email; usually 15–60 min)
5. Verify in the TestFlight app on your phone that the new iOS build is downloadable
6. Open **Releases** here → the open draft already exists (the backend put it there). It already has:
   - `shift-device-connect.apk` attached as an asset
   - `version.json` with the correct `iosBuildNumber` + `androidVersionCode` filled in
7. Edit the draft: set a tag (e.g. `v1.0.0-build5`, doesn't matter to the backend), write release notes
8. Click **Publish release**

The moment you publish, GitHub fires a webhook to the backend. The backend reads `version.json` from the published release's assets, stores the per-platform build numbers as the new "latest", and the operator app's "App update available" banner fires for any user on an older build.

### What the auto-draft contains

`version.json` (auto-written by the backend):

```json
{
  "version": "1.0.0",
  "iosBuildNumber": 24,
  "androidVersionCode": 5
}
```

`version` is informational (the project's semver in `app.json` is permanently `"1.0.0"` and isn't bumped per release). The banner check on the operator app compares the user's installed iOS `buildNumber` or Android `versionCode` against the published value, per platform.

### Why a draft at all (vs. auto-publishing)

The draft exists so **you** decide when users are told there's an update. Until you publish, users see nothing — even though the EAS build is complete. This lets you wait for TestFlight to finish processing, do any last-minute QA, write proper release notes, etc., before flipping the switch.

### Only one open draft at a time

The backend creates a new draft only when there is no open draft. Subsequent EAS builds (e.g. you rebuild Android after a fix) **update the existing draft in place**: the APK gets replaced, `androidVersionCode` in `version.json` gets bumped. Publishing the draft closes it. The next EAS build then starts a fresh draft.

If you ever want to discard an in-progress draft (e.g. you decide a release shouldn't ship), just delete the draft in the GitHub Releases UI. The next EAS build creates a new one.

### Setup the backend needs (one-time, already done if you're reading this in production)

For the EAS-to-draft pipeline to work, the backend has these environment variables set:

- `EAS_WEBHOOK_SECRET` — HMAC secret shared with EAS (via `eas webhook:create`)
- `DISTRIBUTION_WEBHOOK_SECRET` — HMAC secret configured on this repo's GitHub webhook
- `GITHUB_DISTRIBUTION_PAT` — Personal Access Token with `repo` scope on this repo, so the backend can create/edit drafts and upload assets

And:

- EAS webhook: `eas webhook:create --event BUILD --url <backend>/webhooks/eas-build --secret $EAS_WEBHOOK_SECRET`
- GitHub webhook on this repo: Settings → Webhooks → Add webhook
  - Payload URL: `<backend>/webhooks/distribution-release`
  - Content type: `application/json`
  - Secret: `$DISTRIBUTION_WEBHOOK_SECRET`
  - Events: only "Releases"
