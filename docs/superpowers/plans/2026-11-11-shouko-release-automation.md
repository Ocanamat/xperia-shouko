# Shouko Release Automation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the working sync/build workflow into a signed, retryable GitHub Release pipeline compatible with Obtainium.

**Architecture:** A single workflow owns synchronization, release detection, signing, build verification, artifact retention, changelog creation, and publishing. The latest release body records the upstream SHA used for deterministic retry decisions.

**Tech Stack:** GitHub Actions, Bash, Gradle/Android Gradle Plugin, Java keytool, GitHub CLI

**Spec:** `docs/superpowers/specs/2026-11-11-shouko-release-automation-design.md`

## Global Constraints

- Preserve merge-based upstream synchronization and the `master` branch.
- Never print signing secrets or commit a keystore.
- Normal releases only; one `.apk` asset with a stable filename prefix.
- Manual dispatch always builds; schedules build only when current upstream is unreleased.

## Review Focus

- No prior release: publish an initial release rather than skipping.
- Failed prior build after merge: retry the same upstream SHA on the next schedule.
- Latest release body missing/malformed SHA marker: rebuild safely.
- Missing signing secret: fail clearly without leaking values.
- Changelog base not in upstream history: fall back to an initial-release note.

---

### Task 1: Release workflow

**Files:**
- Modify: `.github/workflows/sync-and-build.yml`

**Interfaces:**
- Consumes: `upstream/master`, GitHub release API, four signing secrets.
- Produces: signed release APK, Actions artifact, release tag/body marker.

- [ ] Write and run static workflow assertions that fail against the current file.
- [ ] Replace change-only gating with unreleased-upstream detection plus manual forcing.
- [ ] Configure persistent signing and monotonic version metadata.
- [ ] Build and cryptographically verify the release APK.
- [ ] Generate upstream-only release notes and publish the release.
- [ ] Validate YAML/action semantics and rerun static assertions.

### Task 2: Signing identity and live verification

**Files:**
- Create locally: encrypted signing-key backup under the user-facing outputs folder.
- Configure remotely: `SIGNING_KEY`, `ALIAS`, `KEY_STORE_PASSWORD`, `KEY_PASSWORD`.

**Interfaces:**
- Consumes: the Task 1 secret names and workflow dispatch.
- Produces: reusable Android signing identity and a verified GitHub Release.

- [ ] Generate a long-lived Android signing key and encrypted backup.
- [ ] Upload the four repository secrets without printing their values.
- [ ] Commit and push the workflow.
- [ ] Dispatch and watch the workflow through release publication.
- [ ] Download the release APK and verify its signer matches the backup key.
- [ ] Confirm the release asset and URL shape required by Obtainium.
