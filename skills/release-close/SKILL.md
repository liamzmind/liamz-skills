---
name: release-close
description: Audit and close a mobile app update (Android, iOS, or both) before publishing to Google Play and App Store Connect. Runs in gated phases with a mandatory stop between audit and any change: audit the release diff, apply only approved fixes, then close the release (version check, release notes, fresh signed build, safe staging and commit). Never uploads, submits, tags or pushes without separate approval. Use only when the user explicitly asks to close, finalize or prepare a mobile app release or update for the stores.
---

# Release close

Close one Android, iOS or combined app update through an audit gate, approved fixes, and a release close. Preserve the user's work and never skip the stop between audit and changes.

Run only when the user explicitly asks to close or prepare a release. General requests to test, commit, write a changelog or push a build do not count.

## Phase 0: Setup

1. **Find the project's release configuration.** Look for a release section in `CLAUDE.md`, `AGENTS.md`, `README`, or a `.release-close.md` at the repo root. Use the template in [references/checklists.md](references/checklists.md) to see what is needed: version files and fields, build and test commands, CI workflow names, release-notes location and languages, commit message format, and the files that must never be staged. If anything is missing, ask the user once, then offer to save the answers to `.release-close.md`.
2. **Decide the scope:** Android, iOS or both. Infer it from the request and the changed paths; ask one short question only if both remain plausible. Platforms have independent release boundaries even when their version names match.
3. **Resolve each platform's previous release boundary without guessing.** Prefer, in order: a verified release tag reachable from HEAD, a release commit that changed the version, a verified store or TestFlight build SHA. Cross-check against the changelog and history. For iOS, a marketing version alone is not enough, since many builds can share it. If evidence conflicts or several boundaries are plausible, list the candidates and stop until the user picks one.

## Phases and the state machine

1. A new invocation always runs Phase 0, then Phase 1.
2. Approval of specific findings enters Phase 2 only.
3. After an audit with no unresolved findings, explicit approval such as "proceed" enters Phase 3. State any verification still pending.
4. Never combine Phase 1 and Phase 3 in one turn. Stop after the audit even when it finds nothing.

## Phase 1: Audit and stop

Do not edit source, resources, changelog, history or tags. Running tests and builds is fine.

**Scope of review:** every committed change since each included platform's boundary, plus all staged, unstaged and untracked changes. Treat commits pushed during development as work to audit, not as approval to release.

**Review for:** regressions, crashes, invalid states and ignored error results; persistence, migrations and data safety; background work, permissions and notifications; lifecycle, rotation, accessibility, text scaling, right-to-left layout and small screens where touched; resource and string references and localization consistency; build and manifest compatibility; secrets or debug code left in; generated or shared files kept in sync; tests that cover the important fixes. Use the platform checklists in [references/checklists.md](references/checklists.md).

**Verify, do not assume.** Run the project's own checks: version consistency across files, lint, unit tests, and a release build. Do not count an unrun check as passing. If a required step is blocked (sandbox, missing tool, no macOS), report it as unverified. Do not drive a device, emulator or simulator interactively unless the user authorizes it; list manual visual and interaction checks as unverified instead. Never claim a CI pass for a different source state than the one tested.

**Report and stop** with this structure:

```
Audit: <platform(s) and versions> against <boundary per platform>
Findings:
- [severity] <what, where (path:line), evidence>   (or "None")
Suggested fixes:
- <smallest correction>   (or "None")
Verification:
- Android: <checks and results>
- iOS: <checks, CI run/SHA/result>
- Manual: <verified or unverified>
Status: waiting for approval to fix findings or proceed with closing
```

Sort by severity. Separate proven defects from risks and unverified behavior. Make no corrections, staging, commits or tags before the user responds.

## Phase 2: Apply approved fixes

1. Apply only the fixes the user approved.
2. Reproduce each bug with a failing regression test where practical.
3. Keep unrelated changes untouched and avoid refactors.
4. Run the affected tests, then repeat Phase 1 for every included platform, report what remains, and stop again.

Approval of fixes is not approval to close.

## Phase 3: Close

Enter only after an audit with no unresolved findings and explicit approval to proceed.

1. **Versions.** Confirm the version name and the build or version code are correct and consistent in every place they appear (app and extensions or widgets, all targets). Android's version code must be higher than the last uploaded one. iOS's build number is usually assigned by CI or the build number setting; do not confuse it with the marketing version.
2. **Changelog and release notes.** Derive them only from the verified diff and approved fixes. Never invent changes. Write user-visible changes only, in the app's and store's languages, grouped into broad themes: omit tests, refactors, implementation detail and tiny visual tweaks. Keep the in-app changelog (if any) short and archive older entries in condensed form. Check store length limits in [references/checklists.md](references/checklists.md). Produce a separate title and notes per platform when their changes differ, and one shared copy when they are identical. Do not imply a feature shipped on a platform where it did not.
3. **Final verification.** From a clean state, rerun lint, tests and a fresh release build so no artifact is stale. Confirm the Android bundle is signed with the release key (use the signing tool's verify output, not just file presence). For iOS, require a build and tests on the exact candidate source; if the source changed after the last CI run, report iOS verification as pending. Run `git diff --check`.
4. **Stage and commit safely.**
   - Inspect `git status --short` and classify every path. Stage intended files explicitly; never use `git add -A` before classifying.
   - Never stage credentials, keystores or key files, signing config, local config, build directories, installable artifacts (APK, AAB, IPA, archives, debug symbols), provisioning profiles, derived data or generated projects. See the blocklist in the checklist file.
   - Inspect the staged name-status, the full staged diff and the whitespace check. Block the commit if an intended file is missing or an unexpected one is staged.
   - Commit on the active release branch using the project's message format. Do not amend, do not create empty release commits, do not touch the main branch until the user approves integration.
5. **Final output**, nothing more: the commit SHA or verified source SHA, scope and working-tree state; per platform, the artifact path, size, SHA-256 and signing result (only for artifacts that actually exist); iOS CI run, SHA and result, and anything still pending; the exact changelog entries written; the release title and notes; and any blocker that stopped the close.

**Approval to close does not authorize** a push, a tag, a merge to main, an upload to Play Console or App Store Connect or TestFlight, a store submission, or a hosted release. Each needs its own explicit request. Treat any CI workflow that uploads to a store as a publishing action, never a routine check.

## Failure rules

- Never claim success from earlier results; rerun the final checks.
- Never choose an ambiguous release boundary silently.
- Never fix an audit finding before approval.
- Never present unverified visual or device behavior as verified.
- Never print or commit secrets, keystores or passwords, and never put them in release notes or logs.
- Never require a version bump on one platform because the other is releasing.
- Never publish, upload or submit as part of closing.
