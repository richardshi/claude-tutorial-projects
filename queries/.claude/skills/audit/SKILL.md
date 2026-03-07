---
name: audit
description: Fix vulnerable npm dependencies. Use when the user asks to audit, fix security vulnerabilities, or update vulnerable packages.
disable-model-invocation: true
---

Your goal is to update any vulnerable dependencies.

Do the following:
1. Read `known-issues.md` (in the same directory as this file) to load the list of previously investigated, unfixable issues
2. Run `npm audit` to find vulnerable installed packages in this project
3. For each reported vulnerability, check if it matches a known issue in `known-issues.md`. If it matches and the advisory ID and affected range are the same, skip re-investigating it and note it as a known issue in your report
4. For any vulnerability NOT in `known-issues.md`, attempt to fix it using this decision tree:
   a. Run `npm audit fix` first — this fixes vulnerabilities without breaking changes
   b. If vulnerabilities remain, check whether they are **build-time only** (e.g. inside `node-gyp`, `node-pre-gyp`, or other install-time tooling). These only run during `npm install`, not at runtime.
   c. For build-time vulnerabilities where `--force` would cause breaking changes: add an `"overrides"` block to `package.json` to pin the vulnerable transitive package to a safe version. Example:
      ```json
      "overrides": {
        "tar": ">=7.5.10",
        "@tootallnate/once": ">=3.0.1"
      }
      ```
      Then run `npm install` to apply overrides, and verify with `npm audit`.
   d. For runtime vulnerabilities where no non-breaking fix exists: try upgrading the direct dependency that pulls in the vulnerable package, or add an override if the API is compatible.
   e. If a fix is truly impossible (e.g. no patched version exists, or upgrade breaks the app), document it in `known-issues.md`.
5. If a fix is found for a previously known issue, remove or update its entry in `known-issues.md`
6. Run `npm audit` once more to confirm 0 vulnerabilities (or only known issues remain)
7. Summarize: what was fixed (and how), what remains unfixed and why, and what known issues were skipped

## Key lessons

- **`npm audit fix --force` downgrades packages** — avoid it if the suggested downgrade is to an older major version. Use overrides instead.
- **Build-time vs runtime**: vulnerabilities in `node-gyp`/`node-pre-gyp` dependency chains (e.g. `tar`, `minimatch`, `cacache`, `http-proxy-agent`) are only exploitable if an attacker can influence the build environment. They are lower risk than runtime vulnerabilities, but still worth fixing via overrides.
- **`package-lock.json` resolved URLs**: if a package resolved URL points to a private registry (e.g. Artifactory) and you get E401, update the `resolved` field in `package-lock.json` to point to `https://registry.npmjs.org/` and re-run `npm install`.
