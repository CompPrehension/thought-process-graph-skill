---
name: thought-process-graph
description: Use and develop CompPrehension thought process graphs/trees (TPG) and LOQI domain models. Execute graphs and queries via MCP/CLI, interpret results, author or validate models, debug reasoning, and investigate its_DomainModel, its_Reasoner, its_QuestionGen, DomainSolvingModel, or the TPG methodology.
---

# Thought Process Graph

Use this skill for CompPrehension ITS work involving thought process graphs (`tpg`), LOQI domain models, and the local `its_DomainModel`, `its_Reasoner`, and `its_QuestionGen` projects.

All shell examples in this skill are written for bash. If the session runs in PowerShell or cmd, translate them (quoting, `find` vs `Get-ChildItem`, path separators) instead of pasting them verbatim.

## Rationale

Thought Process Graphs make the reasoning taught by an intelligent tutoring system explicit: questions, object searches, checks, and conclusions over a task situation. A graph usually captures the reasoning substeps for performing or verifying one problem-solving step. Together with the domain's concepts, properties, and relationships, it provides an executable model that can be reused across task instances.

Solving/hinting graphs find correct actions and explain them. Verification graphs check the student's proposed action against domain constraints, identifying the violated rule and the concrete circumstances of an error. This supports explanatory feedback and pedagogical questions that help the student locate a gap in their reasoning. The structure matters educationally: unnecessary ordering of independent checks can make a tutor reject valid reasoning paths.

The approach aims to reduce the effort of authoring formal domain models while preserving meaningful reasoning and feedback. Its demonstrated scope is relatively simple intellectual skills governed by well-defined rules. The paper-based methodology and its limits are summarized in [references/modeling-principles.md](references/modeling-principles.md).

## Start Here

1. **Do the smallest thing that answers the request.** Pick the route below that matches the requested outcome and add others only when the task needs them. Running a graph does not call for reviewing its design, and explaining a concept does not call for reading sources. If the user limits tools, sources, or file changes ("only MCP", "don't run anything", "don't edit the model"), stay within that and mention what you could not check. Those limits are usually there to keep the work fast and predictable. A stated preference ("prefer MCP") still allows a sensible fallback. When the user gives a specific call or command to run, run that one; extra helper calls (CLI help, validation, environment searches) are worth adding only if the given call cannot succeed without them.
2. **Use the supplied context.** Follow model locations, paths, repository links, and dependency versions already provided by the user or project. Running a supplied model through an available MCP tool needs no source or version discovery.
3. **Execute through MCP first, CLI second.** Prefer the CompPrehension MCP operations and call them using their current descriptions and input schemas. Logical names such as `reasoner__reason` or `domain__validate-dsm` usually carry a client-specific prefix. Some clients load MCP tool schemas lazily, so search the available or deferred tools for `reasoner`/`domain` before concluding MCP is missing. Use whatever version information the server provides; there is no need to look for a version before an ordinary call. Without MCP, use the CLI via [references/cli-guide.md](references/cli-guide.md); a missing MCP tool is a reason to switch to the CLI, not to start setting up or diagnosing the server, unless the user asks for that.

For LOQI syntax work, start with the header and relevant rules in [assets/LoqiGrammar.g4](assets/LoqiGrammar.g4). Reading the grammar is for understanding or writing LOQI, not for passing supplied input to a tool. If the snapshot's version differs from the target, or the parser disagrees with it, [references/source-discovery.md](references/source-discovery.md) shows how to find the matching grammar and builders.

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

1. Prefer explicit references over memory. Nothing about LOQI, TPG, or the `its_*` projects is in training data in a reliable form, so any syntax or behavior you "remember" is a guess. Check syntax against the bundled grammar, tool usage against the tool schema or CLI help.
2. When the question is "what happens with this input", run the tool: validation, conversion, and reasoning are cheap and give the actual answer. For "how is this designed or implemented", read the wiki first and the sources only when the wiki is not enough (see the `implementation` route).
3. Treat CompPrehension tools and projects as potentially buggy. When something fails during a diagnosis, separate incorrect usage from a likely project defect; a request that is only about getting a result can simply report the failure.
4. When reporting a suspected project bug, include the runtime version, the exact call or command, the full error text, the inputs or a minimal reproducer, and the stage (parser, builder, validator, reasoner) that appears to fail.

## What To Open For Which Task

Pick by the requested outcome; this is a menu, not a pipeline. Read only the relevant sections of the selected references.

| Route | Requested outcome | First reference / action |
|---|---|---|
| `loqi` | Read, explain, or edit graph/domain text and expressions | [Bundled grammar](assets/LoqiGrammar.g4), including its version header; [loqi-tpg-reference.md](references/loqi-tpg-reference.md) for compact orientation. |
| `execute` | Apply an existing TPG to a situation, obtain a decision or values, run an expression query, interpret the answer | [model-execution.md](references/model-execution.md); call `reason` / `expression-query` via MCP or CLI. |
| `model` | Create a graph, add new reasoning or domain data, or methodologically review a graph | [modeling-principles.md](references/modeling-principles.md), then grammar/tooling as needed. Syntax-only lookup does not need this guide. Methodological findings about existing parts are proposals for the user to approve, not edits to apply. |
| `validate` | Check syntax, types, model loading, or conversion validity | Relevant grammar rules and `validate-dsm`, `validate-domain-loqi`, or `tree-loqi-to-xml`; [cli-guide.md](references/cli-guide.md) for CLI parameters. Methodological review is the `model` route. |
| `convert` | Convert graph/domain representations, inspect XML, or locate a node by metadata | Conversion / `discover-tree` tools; [xml-analysis.md](references/xml-analysis.md) for XML structure and correlation. |
| `trace` | Interpret an existing trace or explain the path producing a result | [trace-analysis.md](references/trace-analysis.md). |
| `debug` | Diagnose unexpected results, expression behavior, or execution failures | [debugging.md](references/debugging.md) (MCP and CLI), then trace analysis. |
| `explain` | Explain the concept and its pedagogical rationale | Rationale above; [modeling-principles.md](references/modeling-principles.md) only for deeper modeling choices. |
| `implementation` | Understand behavior or investigate `its_DomainModel`, `its_Reasoner`, or `its_QuestionGen` | [docs-workflow.md](references/docs-workflow.md) for the relevant wiki pages; matching project sources if the wiki is not enough. |
| `research` | Deep scientific or pedagogical investigation, citations, original examples, or empirical evidence | Original articles via [papers.md](references/papers.md). They do not establish current syntax or runtime behavior. |

[source-discovery.md](references/source-discovery.md) is for finding matching sources, the grammar, or local CLI jars; [cli-guide.md](references/cli-guide.md) is for CLI flags, output formats, and scripting. Neither is needed to call an available MCP tool. [mcp-setup.md](references/mcp-setup.md) is for installing or connecting the server when the user asks, and for authentication or file-access errors returned by an MCP call; an ordinary call, a missing tool, or an unexpected graph conclusion does not need it.

## Project Orientation

If work is already happening inside one of the skill's target projects listed below, use that workspace's sources for implementation analysis and version checks before searching other checkouts or upstream. For LOQI syntax lookup, start with the bundled grammar as described above; matching project sources resolve incompatibilities.

- `its_DomainModel`: domain model, LOQI parser/writer, tree LOQI/XML conversion, RDF conversion, DomainSolvingModel validation.
- `its_Reasoner`: graph/tree-based reasoning over a `DomainSolvingModel` plus a specific LOQI domain; includes expression-query support and JSONL output for automation.
- `its_QuestionGen`: pedagogical (auxiliary) question generation from graphs. This skill bundles no recipes for it on purpose. When a task needs it, research it yourself: the wiki section `3. Наводящие вопросы (its_QuestionGen)` (see [references/docs-workflow.md](references/docs-workflow.md)) and the `its_QuestionGen` sources or source jar (see [references/source-discovery.md](references/source-discovery.md)). Do not assume its API from the other two projects.
- GitHub organization for upstream repositories: https://github.com/CompPrehension/

## Validation Habits

Use these when you change or produce a model. If the user asked only to run, analyze, or draft something, mention useful checks instead of running them.

- Validate domain/model changes with `validate-dsm` / `validate-domain-loqi`.
- Validate tree LOQI by converting it with `tree-loqi-to-xml`, preferably with a model directory and tag when a model context exists.
- When starting from XML, use `decompile-tree` for inspection or migration help, but verify the produced TPG before treating it as authoritative.
- To locate nodes by metadata in compiled XML, a large graph, or by `line`, use `discover-tree`; in a small `.tpg` a text search is enough.
- When starting from legacy dictionaries/TTL inputs, use `dict-to-loqi` to generate a LOQI model directory before deeper analysis or conversion work.
- Run `reason` when the task asks whether a graph/tree actually solves a situation. Operation names here use the execution tool selected above; MCP arguments come from its schema, CLI arguments from the CLI guide.
- Choose human or structured (JSONL / JSON) reasoner trace output based on the analysis task.
- In TPG/LOQI tree code, watch for `merge` statements. They merge the continuation of the current branch; a `merge` without a following continuation is an error in the current builder logic.
- If CLI execution is needed but no runnable jar is present, follow [references/source-discovery.md](references/source-discovery.md) for the checkout fallback; ask for an artifact or build/install only if no usable local option exists.
