# Release-close checklists

## 1. Project release configuration (template)

Keep this in `.release-close.md`, or a release section of `CLAUDE.md` or `AGENTS.md`. Ask the user for anything missing.

```
Platforms: android / ios / both
Repo layout: (paths to the Android and iOS projects)

Versions
  Android: file, versionName field, versionCode field
  iOS: file or setting for marketing version (all targets: app, widgets, extensions)
  iOS build number: assigned by (CI / manual)
  Version name scheme: (e.g. same on both platforms, or independent)

Commands (run from where?)
  Lint:
  Unit tests:
  Release build (Android bundle):
  iOS build and tests (local macOS or CI workflow name):
  Signing verification (command):
  Generated or shared files and the command that refreshes or checks them:

Release notes
  In-app changelog location (or none):
  Languages:
  Store notes format (title? length?):

Git
  Release branch convention:
  Commit message format (Android / iOS / combined):
  Main branch policy: (do not touch until approved)

Never stage (add project-specific paths):
Never run as a check (CI workflows that upload to a store):
```

## 2. Android audit checklist

- Version code is higher than the last uploaded build; version name matches the plan.
- Target and compile SDK meet the current Play Console requirement (check the Play Console policy page for the current deadline; do not assume).
- Manifest: new permissions are justified and declared with the right attributes; exported components are intentional; no debuggable flag in the release build.
- Release build type: minification or shrinking config matches the last release; no keep-rule regressions that strip needed classes.
- Data safety: any new data collection, SDK or permission is reflected in the Play data safety form and privacy policy.
- Storage, preferences and database migrations: old installs upgrade without data loss; a migration test exists where the schema changed.
- Background work, alarms and notifications: behavior under battery limits and the permission model of newer Android versions.
- Resources: no missing string, drawable or locale entries; right-to-left layout and text scaling where touched.
- Release bundle (AAB) is built fresh, signed with the release key, and signature verified with the signing tool.

## 3. iOS audit checklist

- Marketing version is correct and matches in the app and all extensions or widgets. The build number is unique for the version (usually CI-assigned).
- Privacy manifest and "required reason" API declarations are present and current for the app and any SDKs; App Privacy answers still match behavior.
- Entitlements and capabilities match what the code uses (push, app groups, associated domains, widgets).
- Info.plist usage-description strings exist for every permission requested, with clear user-facing text.
- Permissions and notification flows, deep links, widgets and background modes where touched.
- Layout on small devices, large text sizes and right-to-left where touched.
- Signing: distribution certificate and provisioning profile are valid and not committed. A signed build is a separate publish artifact.
- Build and tests ran on the exact candidate source (CI or local macOS). A run on a different commit proves nothing about this one.

## 4. Store release-notes limits

Check the current limits before writing, since stores change them. As a guide: Google Play release notes are limited to about 500 characters per language; App Store Connect "What's New" is limited to about 4,000 characters. Keep notes short, user-facing and free of implementation detail. Provide notes in every language the listing supports, or state which are missing.

## 5. Never stage

Credentials and keys (keystores, `*.p12`, `*.p8`, `*.pem`, service-account JSON), signing and local config (`keystore.properties`, `local.properties`, `.env`), build output (`build/`, `.gradle/`, derived data), installable artifacts (`*.apk`, `*.aab`, `*.ipa`, `*.xcarchive`, `*.dSYM`), provisioning profiles (`*.mobileprovision`), and generated project files that the project regenerates. Ordinary source, tests, manifests, privacy manifests, entitlements, project definitions and committed generated resources stay eligible.

## 6. Final report checklist

- Source SHA, scope and working-tree state.
- Per platform: artifact path, size, SHA-256, signing result, for artifacts that exist only.
- iOS: CI run, SHA, result, pending items.
- Changelog entries written and release title and notes.
- Anything unverified, and any blocker.
