---
name: thought-process-graph
description: Work with CompPrehension thought process graphs/trees, LOQI domain and tree files, and the its_DomainModel, its_Reasoner, and its_QuestionGen projects. Use when a task mentions thought process graph, thought process tree, TPG, LOQI, its_DomainModel, its_Reasoner or their CLI, DomainSolvingModel, validation/conversion of LOQI/XML/RDF models, or auxiliary question generation in CompPrehension ITS projects.
---

# Thought Process Graph

Use this skill for CompPrehension ITS work involving thought process graphs (`tpg`), LOQI domain models, and the local `its_DomainModel`, `its_Reasoner`, and `its_QuestionGen` projects.

## Core Workflow

1. Prefer current local sources over memory. If the task involves implementation details, CLI behavior, or grammar correctness, inspect the relevant source first.
2. Treat CompPrehension tools and projects as potentially buggy. If a command fails or throws, determine whether the agent used it incorrectly or the project itself likely has a bug.
3. If it looks like a project bug, collect diagnostics before concluding: exact artifact/version used, full command line, full stack trace/error text, relevant input files or a minimal reproducer, and the source location or builder/validator stage that appears to fail.
4. For user-facing TPG concepts and syntax, use the project documentation workflow in [references/docs-workflow.md](references/docs-workflow.md). Clone the current docs into a temporary project folder and search the local copy instead of relying on memory.
5. For scientific/background claims, use `assets/kii-2023-thought-process-graph.pdf` as the bundled article. Do not load it unless the task asks for theory, citations, terminology, or paper-level context.
6. For CLI usage, read [references/cli-guide.md](references/cli-guide.md).
7. For LOQI/TPG syntax and grammar-sensitive code, read [references/loqi-tpg-reference.md](references/loqi-tpg-reference.md) and verify against the latest packaged `LoqiGrammar.g4` from current sources or source jars.
8. For trace analysis, expression traces, and trace-to-TPG/LOQI matching, read [references/trace-analysis.md](references/trace-analysis.md).
9. For compiled XML tree structure and XML-to-TPG/LOQI matching, read [references/xml-analysis.md](references/xml-analysis.md).
10. For finding jars, source jars, grammar files, and project code, read [references/source-discovery.md](references/source-discovery.md).
11. For debugging thought process graphs/trees or LOQI expressions, read [references/debugging.md](references/debugging.md).

## Project Orientation

If work is already happening inside one of the skill's target projects listed below, orient primarily around that project's code as opened in the current workspace. Treat those workspace sources as the first authority before searching sibling checkouts, bundled references, upstream repositories, or memory.

- `its_DomainModel`: domain model, LOQI parser/writer, tree LOQI/XML conversion, RDF conversion, DomainSolvingModel validation.
- `its_Reasoner`: graph/tree-based reasoning over a `DomainSolvingModel` plus a specific LOQI domain; includes expression-query support and JSONL output for automation.
- `its_QuestionGen`: auxiliary question generation. Use it only when the task needs generated/helper questions; inspect its local/Maven sources before assuming APIs.
- GitHub organization for upstream repositories: https://github.com/CompPrehension/

## Validation Habits

- Validate domain/model changes with `domain-cli validate-dsm` or `domain-cli validate-domain-loqi` when possible.
- Validate tree LOQI by converting it with `domain-cli tree-loqi-to-xml`, preferably with `--model-dir` and `--tag` when a model context exists.
- When starting from XML, use `domain-cli decompile-tree` for inspection or migration help, but verify the produced TPG before treating it as authoritative.
- When starting from legacy dictionaries/TTL inputs, use `domain-cli dict-to-loqi` to generate a LOQI model directory before deeper analysis or conversion work.
- Run `reasoner-cli reason` when the task asks whether a graph/tree actually solves a situation.
- Choose human or JSONL reasoner trace output based on the analysis task; read [references/cli-guide.md](references/cli-guide.md) for current trace and timeout flags before relying on them.
- In TPG/LOQI tree code, watch for `merge` statements. They merge the continuation of the current branch; a `merge` without a following continuation is an error in the current builder logic.
- If a CLI jar is not present locally, ask the user to install/build the corresponding artifact into the local Maven repository or provide the jar.
