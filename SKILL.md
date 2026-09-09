---
name: thought-process-graph
description: Work with CompPrehension thought process graphs/trees, LOQI domain and tree files, and the its_DomainModel, its_Reasoner, and its_QuestionGen projects. Use when a task mentions thought process graph, thought process tree, TPG, LOQI, its_DomainModel, its_Reasoner or their CLI, DomainSolvingModel, validation/conversion of LOQI/XML/RDF models, or auxiliary question generation in CompPrehension ITS projects.
---

# Thought Process Graph

Use this skill for CompPrehension ITS work involving thought process graphs (`tpg`), LOQI domain models, and the local `its_DomainModel`, `its_Reasoner`, and `its_QuestionGen` projects.

All shell examples in this skill are written for bash. If the session runs in PowerShell or cmd, translate them (quoting, `find` vs `Get-ChildItem`, path separators) instead of pasting them verbatim.

## Glossary

Read this once if the terms are new; the rest of the skill assumes them.

- **Thought process graph (TPG)**: a graph of reasoning steps that solves or verifies one problem-solving step over a knowledge graph of the task. Older names: decision tree, thought process tree. The code still uses `DecisionTree*` class names. Written in LOQI2 (`tpg Name(...) { ... }` in a `.tpg` file) or in compiled XML.
- **LOQI**: the text language for domain models (classes, enums, properties, relationships, objects) and for expressions. **LOQI2** is the tree syntax on top of it. One grammar, `LoqiGrammar.g4`, covers both.
- **Domain model** (`DomainModel`): the declarations of a subject domain, plus, in a *situation*, concrete objects. Lives in `domain.loqi`.
- **Tag** (`tag_<name>.loqi`): an optional part of the domain model specific to a class of situations (for example a programming language). It is merged into the base model on demand; the CLI selects it with `--tag`.
- **Domain solving model** (`DomainSolvingModel`, "model directory"): a directory holding `domain.loqi`, any `tag_*.loqi`, and graph files (`<name>.tpg`, `tree_<name>.xml`, `tree.xml` for the unnamed default graph). Scanning is not recursive and files are recognized by name only.
- **Specific domain / situation** (`DomainModel` again, often `situation.loqi` or "specific LOQI"): the concrete learning task as objects and relationships. It is not part of the model directory; it references the model and is merged with it before reasoning. `reasoner-cli reason MODEL_DIR SPECIFIC_LOQI` takes both.
- **Learning situation** (`LearningSituation`, `its_Reasoner`): the merged situation plus the graph's input variables, ready for interpretation.
- **Trace**: the executed path through the graph returned by the reasoner, with a branch result (`correct`, `error`, ...) and final variables. Available as human text or JSONL events.
- **Metadata**: key/value annotations on nodes, branches, and outcomes (`id`, `line`, `skill`, `exception`, localized `label`), used to correlate traces, XML, and source, and by question generation.

## Working Principles

1. Prefer current local sources over memory. Nothing about LOQI, TPG, or the `its_*` projects is in training data in a reliable form, so any syntax or behavior you "remember" is a guess. If the task involves implementation details, CLI behavior, or grammar correctness, inspect the relevant source first.
2. Treat CompPrehension tools and projects as potentially buggy. If a command fails or throws, determine whether the agent used it incorrectly or the project itself likely has a bug.
3. If it looks like a project bug, collect diagnostics before concluding: exact artifact/version used, full command line, full stack trace/error text, relevant input files or a minimal reproducer, and the source location or builder/validator stage that appears to fail.
4. Prefer running a tool over reading code, and reading code over reading documentation, when the question is "what happens": validation, conversion, and reasoning are cheap and give the actual answer.

## What To Open For Which Task

Start from the task, run the tool that answers it, then open the one reference that explains the output. Every reference is self-contained; do not read them all up front.

- **Locate tools and sources first.** Before any command, find the CLI jars (or MCP tools, see below), source jars, and `LoqiGrammar.g4`: [references/source-discovery.md](references/source-discovery.md). Every other reference assumes `DOMAIN_CLI_JAR` and `REASONER_CLI_JAR` from there.
- **Run or script the CLI** (flags, output formats, JSONL events, recipes): [references/cli-guide.md](references/cli-guide.md).
- **Model or tree does not validate / convert**, or the question is "is this LOQI/TPG correct": run `validate-dsm`, `validate-domain-loqi`, or `tree-loqi-to-xml` with a model directory; then [references/loqi-tpg-reference.md](references/loqi-tpg-reference.md) for orientation, and confirm against the current `LoqiGrammar.g4` and builder code, never against memory.
- **Reasoner gives an unexpected result, wrong branch, or exception**: run `reason` with `--verbose` (and `--debug` on crashes), read the trace, then [references/debugging.md](references/debugging.md) for `--debug`, `debug:*` procedures, and where to read reasoner source; [references/trace-analysis.md](references/trace-analysis.md) for the trace structure and how to match trace steps to graph nodes.
- **An expression evaluates unexpectedly**: run `expression-query --trace --verbose` against the situation; same two references.
- **Correlate a compiled XML tree with a `.tpg` or with a trace**, or migrate XML back to LOQI2: `discover-tree` / `decompile-tree`, then [references/xml-analysis.md](references/xml-analysis.md).
- **Find a node by `id`, `line`, `skill`, or other metadata**: `discover-tree`; metadata conventions are in the wiki (see the docs map).
- **Explain a concept, a node kind, LOQI2 syntax, or the model directory layout to the user**: clone the wiki and open the mapped page: [references/docs-workflow.md](references/docs-workflow.md). The docs are in Russian; the map tells you which file answers which question.
- **Theory, citations, pedagogical meaning of node kinds, or an explanation of the approach for a non-developer**: [references/papers.md](references/papers.md) first. It says which of the two bundled articles in `assets/` answers what, gives page maps so you read only the needed pages, and maps paper terminology to current names. The articles predate LOQI2: never take syntax or tool behavior from them.

## MCP Toolchain Server

The CompPrehension toolchain server (repository `CompPrehension-Toolchain-Server`, usually checked out as `compph-toolchain-server`) wraps `its_DomainModel`, `its_Reasoner`, and `meaning_tree` in one long-running JVM and exposes them over MCP at `POST /mcp` (Streamable HTTP). It exists because every CLI call pays JVM start-up; the server pays it once.

Use it only when it is registered in the current agent client as an MCP server and its tools appear in your tool list. The tools are named `<module>__<method>` (clients may add their own prefix, for example `mcp__compph-toolchain-server__domain__validate-dsm`):

| Module | Tools |
|---|---|
| `domain` | `validate-dsm`, `tree-loqi-to-xml`, `decompile-tree`, `discover-tree`, `dict-to-loqi`, `validate-domain-loqi`, `domain-to-rdf`, `rdf-to-domain-loqi` |
| `reasoner` | `reason`, `expression-query` |

How to decide:

1. Look for these tools among your available tools. If they are there, prefer them for routine validation, conversion, and reasoner runs. Read each tool's input schema before calling; parameter names are camelCase (`model`, `domainLoqi`, `treeLoqi`, `jsonTrace`, `timeLimitSeconds`) and differ from CLI flags.
2. If the tools are not present, do not poke the HTTP endpoint with `curl` or hand-written JSON-RPC as a substitute. Use the CLI. If the user wants the server, point them to the server README: build it, run `java -jar target/compph-toolchain-server-<version>.jar [PORT]`, then register `http://localhost:<port>/mcp` as an HTTP MCP server in their client (Claude Code: `claude mcp add --transport http compph-toolchain-server http://localhost:8080/mcp`).
3. File and directory arguments are payloads, not paths: a file is an inline string or `{"text": ...}`/`{"base64": ...}`; a directory is `{"files": {"domain.loqi": ..., "main.tpg": ...}}`. `{"path": "/abs/path"}` works only when the server runs with `LOCAL_FILES_DISCOVERY=true` on the same machine. If a call fails on a path payload, switch to inline content.
4. Results are structured JSON that mirrors the reasoner's JSONL events (`branchResult`, `finalNode`, `variables`, `exceptions`, `trace`, optional `metrics`); the trace-reading guidance in [references/trace-analysis.md](references/trace-analysis.md) applies to them unchanged.

Use the CLI directly when the MCP tools are absent, when the task asks for CLI commands, when reproducing a command-line failure, or when you need behavior the tool schemas do not expose. The CLI references remain the source for flags, artifacts, and fallback/debug workflows.

## Project Orientation

If work is already happening inside one of the skill's target projects listed below, orient primarily around that project's code as opened in the current workspace. Treat those workspace sources as the first authority before searching sibling checkouts, bundled references, upstream repositories, or memory.

- `its_DomainModel`: domain model, LOQI parser/writer, tree LOQI/XML conversion, RDF conversion, DomainSolvingModel validation.
- `its_Reasoner`: graph/tree-based reasoning over a `DomainSolvingModel` plus a specific LOQI domain; includes expression-query support and JSONL output for automation.
- `its_QuestionGen`: pedagogical (auxiliary) question generation from graphs. This skill bundles no recipes for it on purpose. When a task needs it, research it yourself: the wiki section `3. Наводящие вопросы (its_QuestionGen)` (see [references/docs-workflow.md](references/docs-workflow.md)) and the `its_QuestionGen` sources or source jar (see [references/source-discovery.md](references/source-discovery.md)). Do not assume its API from the other two projects.
- GitHub organization for upstream repositories: https://github.com/CompPrehension/

## Validation Habits

- Validate domain/model changes with the MCP `domain__validate-dsm` / `domain__validate-domain-loqi` tools when available, or `domain-cli validate-dsm` / `domain-cli validate-domain-loqi` as a fallback.
- Validate tree LOQI by converting it with MCP `domain__tree-loqi-to-xml` or `domain-cli tree-loqi-to-xml`, preferably with a model directory and tag when a model context exists.
- When starting from XML, use MCP `domain__decompile-tree` or `domain-cli decompile-tree` for inspection or migration help, but verify the produced TPG before treating it as authoritative.
- To locate specific decision tree nodes (e.g. by `id`, `line`, or another metadata key), use MCP `domain__discover-tree` or `domain-cli discover-tree` instead of manually scanning LOQI/XML/TPG source.
- When starting from legacy dictionaries/TTL inputs, use MCP `domain__dict-to-loqi` or `domain-cli dict-to-loqi` to generate a LOQI model directory before deeper analysis or conversion work.
- Run MCP `reasoner__reason` or `reasoner-cli reason` when the task asks whether a graph/tree actually solves a situation.
- Choose human or JSONL reasoner trace output based on the analysis task; read [references/cli-guide.md](references/cli-guide.md) for current trace and timeout flags before relying on them.
- In TPG/LOQI tree code, watch for `merge` statements. They merge the continuation of the current branch; a `merge` without a following continuation is an error in the current builder logic.
- If a CLI jar is not present locally, ask the user to install/build the corresponding artifact into the local Maven repository or provide the jar.
