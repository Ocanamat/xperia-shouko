# Shouko Release Automation Design

## Intent

Maintain a personal, installable Shouko build that follows `ivaniskandar/shouko`, updates safely on Android, and is discoverable by Obtainium.

## Goals

- Merge `upstream/master` into the fork every day without discarding fork-only workflow changes.
- Retry an unreleased upstream commit after a failed build, even when no new commit arrives later.
- Build a release APK with one persistent signing identity stored in GitHub Secrets.
- Publish a normal GitHub Release with a unique UTC date/run tag and an upstream-only changelog.
- Give every APK a monotonically increasing Android `versionCode`.
- Keep manual dispatch as a force-build/release path.

## Non-goals

- Change Shouko application behavior or its package ID.
- Publish to an app store.
- Rewrite upstream history or automatically resolve merge conflicts.

## Approach

Use one scheduled/manual workflow. It merges upstream, compares the current upstream SHA with an `upstream-sha` marker in the latest release, configures the existing Gradle signing support from four GitHub Secrets, builds and verifies a release APK, then publishes it with `gh release create`. This avoids the extra token and coordination required by split sync/build workflows.

The Android `versionCode` is seconds since 2020-01-01, keeping it monotonic and below Android's signed 32-bit ceiling for decades. The GitHub tag and Android `versionName` use the identical value `<base-version>-YYYY.MM.DD-HHMMSS-r<run>-<upstream-sha>` so Obtainium does not report false updates; the value remains human-readable and collision-resistant.

## Constraints and failure behavior

- Merge conflicts fail safely without rewriting the fork.
- Missing or invalid signing secrets fail before compilation.
- A failed build leaves the upstream merge in the fork, but the next schedule retries because no matching release marker exists.
- The first signed APK may require uninstalling the previously installed debug-signed APK.
- The private signing key must be backed up; losing it makes future in-place Android updates impossible.
