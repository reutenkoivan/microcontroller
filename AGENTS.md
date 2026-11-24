# Microcontroller development knowledge base

Use `wiki/` for knowledge about microcontrollers and embedded development. Use `docs/` for the repository's structure, conventions, and workflows. Write wiki content in Ukrainian, preserving established technical names and acronyms. Cite primary sources for technical claims.

When you add, edit, move, or delete wiki material, follow the README contents sync rules in `docs/agents/domain.md` in the same change.

## Agent skills

### Engineering Compass

Engineering Compass release: 0.3.0

Before planning or working in this repository, resolve `$engineering-compass` and read `plugin.json` in the active global package. If the versions match, apply `principles.md` and `context-map.md` from that package to the task and its consequences.

If the repository version is older, report both versions and ask the maintainer to invoke `$engineering-compass-update` here before continuing. If the repository version is newer, report both versions and use the installed `$engineering-compass-setup` procedure to audit local policy and verify the integration before recording the installed version.

An explicit `$engineering-compass-update` can run before the versions match. If it expects Git revision fields, first refresh the clean global Git checkout on `release` with `git pull --ff-only`, then read Update from the refreshed package. Preserve this version-based entry format when applying Setup or Update.

### Issue tracker

Discuss proposed documentation changes in the conversation. See `docs/agents/issue-tracker.md`.

### Triage labels

Read `docs/agents/triage-labels.md` when assigning a role to a proposed documentation change.

### Domain docs

The knowledge base uses a single-context layout. Read `docs/agents/domain.md` when updating wiki terminology or organizing documentation.
