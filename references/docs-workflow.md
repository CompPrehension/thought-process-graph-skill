# Documentation Workflow

Use this workflow whenever a task needs CompPrehension user documentation for thought process graphs, thought process trees, decision trees, or related LOQI examples.

## Primary Documentation

Published user documentation:

```text
https://compprehension.github.io/docs/thought_process_graph/
```

Repository source to clone:

```text
https://github.com/CompPrehension/CompPrehension.github.io/tree/production/docs/decision_tree
```

The documentation source lives in the `production` branch under `docs/decision_tree`.

## Clone Locally Before Reading

When documentation is required and network access is available, clone the current `production` branch into a temporary folder inside the current project workspace, then search/read the local copy.

Preferred location:

```text
.tmp/CompPrehension.github.io-production/
```

Commands:

Linux/WSL/macOS:

```bash
mkdir -p .tmp
git clone --depth 1 --branch production \
  https://github.com/CompPrehension/CompPrehension.github.io.git \
  .tmp/CompPrehension.github.io-production
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force .tmp
git clone --depth 1 --branch production `
  https://github.com/CompPrehension/CompPrehension.github.io.git `
  .tmp\CompPrehension.github.io-production
```

Then search only the relevant docs subtree:

```bash
rg -n "thought|process|graph|tree|tpg|loqi|decision" .tmp/CompPrehension.github.io-production/docs/decision_tree
find .tmp/CompPrehension.github.io-production/docs/decision_tree -maxdepth 2 -type f
```

PowerShell:

```powershell
rg -n "thought|process|graph|tree|tpg|loqi|decision" .tmp\CompPrehension.github.io-production\docs\decision_tree
Get-ChildItem .tmp\CompPrehension.github.io-production\docs\decision_tree -Recurse -File -Depth 2
```

If the clone already exists, refresh it instead of recloning:

```bash
git -C .tmp/CompPrehension.github.io-production fetch origin production --depth 1
git -C .tmp/CompPrehension.github.io-production checkout production
git -C .tmp/CompPrehension.github.io-production reset --hard origin/production
```

The same `git -C` refresh commands work in PowerShell and cmd; use backslashes in paths only if the shell or tab completion requires them.

Use the local files under:

```text
.tmp/CompPrehension.github.io-production/docs/decision_tree/
```

## If Network Or Git Is Unavailable

If cloning fails because network access is unavailable or the repository cannot be reached:

1. Say that the current documentation could not be cloned.
2. Fall back to local project sources, `LoqiGrammar.g4`, CLI help/source, and bundled references.
3. Avoid claiming documentation details as current unless they were read from a local clone or provided by the user.

## Handling The Temporary Clone

- Keep the clone under `.tmp/` or another clearly temporary project-local directory.
- Do not edit documentation files unless the user explicitly asks to modify documentation.
- Do not commit or treat the temporary clone as part of the target project.
- Use `rg` inside `docs/decision_tree` first; open only the specific files needed for the user task.
