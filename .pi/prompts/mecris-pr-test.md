---
description: Dispatch and poll the CI pipeline (Run Complete Test Suite) for a PR; include OKF validation step. References .github/skills/mecris-pr-test/SKILL.md.
---
Test the PR pipeline:

1. Read `.github/skills/mecris-pr-test/SKILL.md` (dispatch workflow, poll CI, verify OKF validation step).
2. Confirm PR passes `.github/workflows/ci.yml` (includes `make okf-validate`).
3. Verify bundle passes (`okf validate knowledge --strict --drift` → 0 errors, 0 warnings, 0 broken links, 0 orphans, 0 stale).
4. Merge only after validation passes (branch-protected: `main` requires PR + passing CI).
