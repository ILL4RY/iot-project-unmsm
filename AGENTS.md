# Spec Kit workflow

This repository uses GitHub Spec Kit (v1.2.0, Codex integration). The generated
workflow commands are kept in `.agents/skills/speckit-<name>/SKILL.md`, as
recorded in `.specify/integrations/codex.manifest.json`.

When a user invokes a Spec Kit command, open the matching command file and
follow it as the active workflow. Support both command spellings below:

- `/speckit.constitution` or `$speckit-constitution`
- `/speckit.specify` or `$speckit-specify`
- `/speckit.clarify` or `$speckit-clarify`
- `/speckit.plan` or `$speckit-plan`
- `/speckit.checklist` or `$speckit-checklist`
- `/speckit.tasks` or `$speckit-tasks`
- `/speckit.analyze` or `$speckit-analyze`
- `/speckit.implement` or `$speckit-implement`
- `/speckit.converge` or `$speckit-converge`
- `/speckit.taskstoissues` or `$speckit-taskstoissues`

For example, `/speckit.specify` maps to
`.agents/skills/speckit-specify/SKILL.md`. Treat any text following the command as
the command's `$ARGUMENTS` value.

Keep `.specify/` (except the machine-local files listed in
`.specify/.gitignore`), `.agents/skills/speckit-*/`, and generated `specs/`
artifacts in version control. Do not commit credentials or other agent-local
state (`.codex/`, anything else under `.agents/`).
