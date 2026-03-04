# Known Audit Issues

Issues investigated and confirmed unfixable at time of writing. Re-check when `npm audit` output changes.

---

## dompurify (via monaco-editor) — moderate XSS

**Advisory:** GHSA-v2wj-7wpq-c8vv
**Affected range:** `>=3.1.3 <=3.3.1`
**Last checked:** 2026-03-04

**Root cause:** `monaco-editor@0.55.1` pins `dompurify@3.2.7` exactly. No patched version of dompurify exists yet (latest is `3.3.1`, still in vulnerable range).

**What was tried:**
- `npm audit fix` — no-op, packages already up to date
- `overrides: { dompurify: "3.1.2" }` — introduced 3 *high* severity vulnerabilities (mXSS, prototype pollution, XSS in `<=3.2.3` range), worse than the original moderate

**Resolution:** No action taken. Reverted overrides. Waiting for dompurify to release `>3.3.1` or for `monaco-editor` to update its pinned dependency.

**Signal to re-investigate:** If `npm audit` no longer reports this advisory, or reports a different dompurify range, investigate whether a fix is now available.

---

## Pre-existing test failures (unrelated to security)

**Last checked:** 2026-03-04

Two categories of pre-existing failures in the test suite (confirmed via `git status` — lockfile unchanged):

1. **React version mismatch** — `react@19.2.4` vs `react-dom@19.1.0` locked in `package-lock.json`. Causes 7 component test suites to fail with "Incompatible React versions".
2. **`jose` auth tests** — `payload must be an instance of Uint8Array` error in `src/lib/__tests__/auth.test.ts`. Likely a jose API version incompatibility in the test setup.

These are not introduced by audit runs and should be fixed separately.
