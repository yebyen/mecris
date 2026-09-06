# Mecris Agent Guide

Mecris is a personal accountability system. The Pi extension exposes its Python backend as native tools.

## Always-on rules

- `main` is PR-protected. Work on a feature branch and merge through a passing PR.
- Use `/status` for a quick live report. It is a deterministic extension command and does not invoke the model.
- Use `/mecris-orient` for full project orientation, `/mecris-plan` before substantial work, `/mecris-archive` when finishing, and `/mecris-pr-test` for PR validation.
- Core Mecris tools are active at startup; load other capabilities with `mecris_load_tools("<capability>")`.
- Mecris tools normally infer the current user from the logged-in credentials. Do not supply `user_id` unless deliberately operating on another account.

## Progressive project knowledge

Durable project knowledge lives in the OKF bundle under `knowledge/`. Load the `okf-agent-memory` skill only when a task needs architectural history, durable decisions, runbooks, or knowledge maintenance.

Useful commands:

```bash
okf search "<topic>" knowledge --limit=3 --json
okf show <concept-id> knowledge
make okf-validate
```

Do not run an OKF workflow for `/status`.

## Source references

- Pi bridge: `.pi/extensions/mecris/README.md`
- Gall workflow definitions: `.github/skills/mecris-*/SKILL.md`
- Cold start and identity fallback: `knowledge/runbooks/agent-bootstrap.md`
- Full project documentation: `README.md` and `docs/`
