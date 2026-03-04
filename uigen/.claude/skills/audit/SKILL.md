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
4. For any vulnerability NOT in `known-issues.md`, attempt to fix it: run `npm audit fix`, try overrides, or upgrade the affected package
5. If a fix is found for a previously known issue, remove or update its entry in `known-issues.md`
6. Run tests and verify
7. Summarize: what was fixed, what remains unfixed and why, and what known issues were skipped
