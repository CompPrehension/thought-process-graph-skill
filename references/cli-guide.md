# CLI Guide

## Table Of Contents

- [its_DomainModel CLI](#its_domainmodel-cli)
  - [validate-dsm](#validate-dsm)
  - [tree-loqi-to-xml](#tree-loqi-to-xml)
  - [decompile-tree](#decompile-tree)
  - [discover-tree](#discover-tree)
  - [dict-to-loqi](#dict-to-loqi)
  - [validate-domain-loqi](#validate-domain-loqi)
  - [domain-to-rdf](#domain-to-rdf)
  - [rdf-to-domain-loqi](#rdf-to-domain-loqi)
- [its_Reasoner CLI](#its_reasoner-cli)
  - [reason](#reason)
  - [expression-query / expr-query](#expression-query--expr-query)
- [Common Recipes](#common-recipes)

Use the latest local source before relying on this guide:

- `its_DomainModel/src/main/kotlin/CLI.kt`
- `its_Reasoner/src/main/kotlin/misc/CLI.kt`

Current local entrypoints:

- DomainModel main class: `CLIKt`
- Reasoner main class: `misc.CLIKt`

Do not assume fixed versions. First discover the runnable jars:

Linux/WSL/macOS:

```bash
DOMAIN_CLI_JAR="$(find its_DomainModel/target "$HOME/.m2/repository" -name 'its_DomainModel-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"
REASONER_CLI_JAR="$(find its_Reasoner/target "$HOME/.m2/repository" -name 'its_Reasoner-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"

java -jar "$DOMAIN_CLI_JAR" --help
java -jar "$REASONER_CLI_JAR" --help
```

Windows PowerShell:

```powershell
$domainRoots = @("its_DomainModel\target", "$env:USERPROFILE\.m2\repository") | Where-Object { Test-Path $_ }
$reasonerRoots = @("its_Reasoner\target", "$env:USERPROFILE\.m2\repository") | Where-Object { Test-Path $_ }
$DOMAIN_CLI_JAR = Get-ChildItem $domainRoots -Recurse -File -Filter "its_DomainModel-*-all.jar" -ErrorAction SilentlyContinue | Sort-Object LastWriteTime | Select-Object -Last 1 -ExpandProperty FullName
$REASONER_CLI_JAR = Get-ChildItem $reasonerRoots -Recurse -File -Filter "its_Reasoner-*-all.jar" -ErrorAction SilentlyContinue | Sort-Object LastWriteTime | Select-Object -Last 1 -ExpandProperty FullName
java -jar $DOMAIN_CLI_JAR --help
java -jar $REASONER_CLI_JAR --help
```

If a runnable jar is absent, use Maven from the project checkout:

Linux/WSL/macOS:

```bash
cd its_DomainModel && mvn exec:java -Dexec.args="--help"
cd its_Reasoner && mvn exec:java -Dexec.args="--help"
```

Windows PowerShell:

```powershell
Push-Location its_DomainModel; mvn exec:java "-Dexec.args=--help"; Pop-Location
Push-Location its_Reasoner; mvn exec:java "-Dexec.args=--help"; Pop-Location
```

Use `$DOMAIN_CLI_JAR` and `$REASONER_CLI_JAR` in PowerShell examples the same way Bash examples use `"$DOMAIN_CLI_JAR"` and `"$REASONER_CLI_JAR"`.

## its_DomainModel CLI

Command name: `domain-cli`.

### validate-dsm

Validate a `DomainSolvingModel` directory containing files such as `domain.loqi`, `tag_*.loqi`, and `tree_*.xml` or `tree_*.loqi`.

```bash
java -jar "$DOMAIN_CLI_JAR" validate-dsm path/to/model-dir
```

Options:

- `--build-method LOQI|...`: build method, default `LOQI`.
- `--debug`: skip the check for `debug:`-namespace procedures in decision trees (allows validating a model that still contains `debug:breakpoint`, `debug:trace`, etc.).

### tree-loqi-to-xml

Convert a LOQI thought process graph/tree to XML. Use `--model-dir` to validate the tree against a model.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  tree-loqi-to-xml path/to/tree.loqi \
  --model-dir path/to/model-dir \
  --tag java \
  --cdata-expressions \
  -o path/to/tree.xml
```

Options:

- `-o, --output XML_FILE`: write XML to a file; otherwise print to stdout.
- `--model-dir MODEL_DIR`: additionally validate the tree against a `DomainSolvingModel`.
- `--tag TAG`: validate against a merged tag domain; only valid with `--model-dir`.
- `--cdata-expressions`: write expressions in XML as CDATA LOQI.

### decompile-tree

Decompile a decision tree from XML back into LOQI/TPG for inspection.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  decompile-tree path/to/tree.xml \
  --tree-name ExprEval \
  -o path/to/tree.loqi
```

Options:

- `-o, --output TPG_FILE`: write LOQI/TPG to a file; otherwise print to stdout.
- `--tree-name NAME`: tree name to use in the generated `tpg` header; default `ExprEval`.

The current source marks this command as experimental and not production-ready. Use it to inspect or bootstrap a tree, then validate the result before trusting it.

### discover-tree

Search decision tree nodes by metadata (LOQI/TPG or XML, auto-detected by file extension).

```bash
java -jar "$DOMAIN_CLI_JAR" \
  discover-tree path/to/tree.loqi \
  --meta id=n1 \
  --meta line=42 \
  --format jsonl
```

Options:

- `-m, --meta KEY=VALUE`: metadata search criterion; repeatable. At least one is required.
- `--union`: combine multiple `--meta` criteria with OR instead of the default AND.
- `--limit LIMIT`: maximum number of nodes to print; if omitted, all matches are printed.
- `--debug`: build LOQI/TPG trees with debug metadata (`line`) even if `line` isn't among the search criteria.
- `--children`: also show each matched node's immediate (depth-1) child nodes.
- `--format human|jsonl`: default `human`.

If `line` is used as a search key, the tree (when built from LOQI/TPG via `TreeLoqiBuilder`) is automatically built with debug metadata so `line` becomes valid, searchable metadata — `--debug` is not required in that case. `line` metadata is never available for trees loaded from XML.

Matching is done by comparing each node's metadata value (its string form) against the given value; a node's metadata may hold multiple entries for the same property name across different localizations (e.g. `label` unlocalized plus `label` for `ru`), all of which are searched and printed.

For each matching node, the output includes the node's concrete type (e.g. `QuestionNode`, `BranchResultNode`) and all of its metadata entries (including localized ones and `line`, when present in the built tree). JSONL output emits a `summary` event (`found`/`shown` counts) followed by one `node` event per printed node, each shaped as `{"type": "node", "nodeType": "...", "metadata": [{"name": ..., "locCode": ..., "value": ...}, ...]}`. This exact `nodeType`/`metadata` shape is reused by `reasoner-cli reason`'s `final-node` JSONL event (see below).

With `--children`, each `node` event additionally carries a `children` field: `{"total": N, "descriptors": [{"id": ..., "line": ..., "skill": ...}, ...]}`. `total` counts every immediate (depth-1) child node reachable from the match; `descriptors` lists only the children that have at least one of `id`/`line`/`skill` metadata (in whichever subset is present) — children with none of those three keys are counted in `total` but omitted from `descriptors`. "Immediate" means one reasoning step away: branch/outcome targets of aggregations, questions, cycles, etc., or the direct redirect target of a `ProcedureCallNode`; `line` in a descriptor is only present when the tree was built with debug metadata (see above). In human output, this prints as an indented `Children: N total` block under the node.

### dict-to-loqi

Build a LOQI model directory from CSV dictionaries plus `domain.ttl`.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  dict-to-loqi path/to/model-dir path/to/output-dir \
  --separate-metadata \
  --separate-class-property-values
```

Expected input directory contents include `enums.csv`, `classes.csv`, `properties.csv`, `relationships.csv`, and `domain.ttl`.

Options:

- `--separate-metadata`: write metadata into separate sections.
- `--separate-class-property-values`: write class property values into separate sections.

The command writes `domain.loqi` and copies discovered decision-tree files into the output directory.

### validate-domain-loqi

Validate an extra/specific domain LOQI file in the context of a `DomainSolvingModel` and optional tag.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  validate-domain-loqi path/to/specific-domain.loqi path/to/model-dir \
  --tag java \
  --print-merged-loqi
```

Options:

- `--tag TAG`: include the tag domain before merging.
- `--print-merged-loqi`: print the merged domain after validation.

### domain-to-rdf

Build a concrete domain and write RDF Turtle.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  domain-to-rdf path/to/model-dir \
  --tag java \
  --domain-loqi path/to/specific-domain.loqi \
  -o path/to/domain.ttl
```

Options:

- `--build-method LOQI|...`: default `LOQI`.
- `--tag TAG`: merge tag domain.
- `--domain-loqi DOMAIN_LOQI`: merge additional domain LOQI.
- `-o, --output TTL_FILE`: write TTL to a file; otherwise print to stdout.
- `--base-prefix PREFIX`: base RDF prefix.
- `--old-nary-compat`: old n-ary relationship representation.

### rdf-to-domain-loqi

Fill a concrete domain from RDF Turtle and save LOQI.

```bash
java -jar "$DOMAIN_CLI_JAR" \
  rdf-to-domain-loqi path/to/model-dir path/to/domain.ttl \
  --tag java \
  --domain-loqi path/to/specific-domain.loqi \
  --separate-metadata \
  -o path/to/domain.loqi
```

Options:

- `--build-method LOQI|...`: default `LOQI`.
- `--tag TAG`: merge tag domain.
- `--domain-loqi DOMAIN_LOQI`: merge additional LOQI before RDF fill.
- `-o, --output DOMAIN_LOQI`: write LOQI to a file; otherwise print to stdout.
- `--base-prefix PREFIX`: explicit RDF prefix.
- `--old-nary-compat`: old n-ary relationship representation.
- `--throw-invalid-meta`: fail on invalid metadata literals.
- `--separate-metadata`: write metadata in separate sections.
- `--separate-class-property-values`: write class property values in separate sections.

## its_Reasoner CLI

Command name: `reasoner-cli`.

### reason

Merge a base/tag domain with a specific LOQI domain, run reasoning, and print a trace/result.

```bash
java -jar "$REASONER_CLI_JAR" \
  reason path/to/model-dir path/to/specific-domain.loqi \
  --tag java \
  --tree main \
  --verbose \
  --time-measure
```

Options:

- `--tag TAG`: base model tag to merge with the specific domain.
- `--tree TREE_NAME`: decision tree name; default is the unnamed tree.
- `--verbose`: print extra trace details, including LOQI for single operators.
- `--debug`: include debug metadata when building `DomainSolvingModel`; also enables partial trace collection so that if reasoning throws a `ReasoningException`, the partial decision tree trace (or partial expression trace) is printed — to `stderr` in human format, or as a `partial-trace`/`partial-expression-trace` JSONL event.
- `--no-trace`: in human output, print only result and variables.
- `-o, --export-domain OUTPUT_LOQI`: save the final specific domain after reasoning, subtracting the base domain.
- `--time-measure`: print preparation and solve times.
- `--time-limit SECONDS`: stop reasoning after the given number of seconds.
- `--format human|jsonl`: default `human`.
- `--json-trace`: with JSONL output, emit structured trace JSON instead of formatted string.

Debugging note:

- When investigating a specific TPG/tree, prefer setting `--time-limit` so a potentially looping tree fails fast instead of hanging indefinitely.
- For production-style runs and clean reasoning-speed measurements, do not enable `--time-limit` by default; use it only when an explicit safety bound is part of the task.

JSONL output emits events such as `result`, `final-node`, `variables`, branch-result exceptions, `trace`, optional metrics, reasoner output messages, and exported domain artifacts.

The `final-node` event carries the decision tree's terminal (`BranchResultNode`) in the same shape as `domain-cli discover-tree`'s `node` events: `{"type": "final-node", "nodeType": "BranchResultNode", "metadata": [{"name": ..., "locCode": ..., "value": ...}, ...]}`. This event is only emitted in `--format jsonl`; human output is unaffected. `line` metadata on the final node is present only when `--debug` was passed to `reason`.

### expression-query / expr-query

Run a LOQI expression query and print matching object names.

Forms:

```bash
java -jar "$REASONER_CLI_JAR" \
  expression-query path/to/domain.loqi 'find X x { ... }'

java -jar "$REASONER_CLI_JAR" \
  expression-query path/to/model-dir path/to/domain.loqi 'find X x { ... }' \
  --tag java --trace --verbose --limit 20 --format jsonl
```

Options:

- `--tag TAG`: only valid with `MODEL_DIR`; merges a base tag with the specific domain.
- `--debug`: include debug metadata when building the model; also prints a partial expression trace if `--trace` is set and the query throws a `ReasoningException`.
- `--trace`: print expression trace.
- `--verbose`: verbose expression trace.
- `--limit LIMIT`: maximum number of object names; non-negative.
- `--loqi`: additionally serialize each found object into LOQI (its `obj name : Class { ... }` declaration, via `DomainModel`'s LOQI writer) and print/emit it alongside the object name.
- `--time-measure`: print query execution time.
- `--time-limit SECONDS`: stop query execution after the given number of seconds.
- `--format human|jsonl`: default `human`.
- `--json-trace`: with JSONL output, emit structured expression trace JSON instead of a formatted string.

With `--loqi`, human output prints each object's LOQI declaration indented beneath its name. JSONL output adds an `objectsLoqi` field to the `expression-query-result` event — `[{"name": ..., "loqi": ...}, ...]`, one entry per object in `objects`, in the same order — leaving the plain `objects` name array unchanged for existing consumers.

## Common Recipes

Validate a model, convert a tree, then run it:

```bash
java -jar "$DOMAIN_CLI_JAR" validate-dsm input_examples_expressions_prod
java -jar "$DOMAIN_CLI_JAR" tree-loqi-to-xml tree.loqi --model-dir input_examples_expressions_prod -o tree.xml
java -jar "$REASONER_CLI_JAR" reason input_examples_expressions_prod specific.loqi --format jsonl --json-trace
```

When automating, prefer `--format jsonl` and parse by the `type` field. In human mode, stack traces are printed on CLI errors; in JSONL mode, errors are emitted as JSON error events.

If older notes or discussions mention `time_limit`, treat that as the current CLI option `--time-limit` unless you have newer local sources showing a different spelling.

If a command crashes, do not assume user error immediately. Record the exact jar/version, command line, stack trace, and minimal input set needed to reproduce the failure, then inspect whether the failure comes from invalid input, a merge/build validation rule, or a project bug.
