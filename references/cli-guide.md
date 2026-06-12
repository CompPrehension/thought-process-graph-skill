# CLI Guide

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
- `--debug`: include debug metadata when building `DomainSolvingModel`.
- `--no-trace`: in human output, print only result and variables.
- `-o, --export-domain OUTPUT_LOQI`: save the final specific domain after reasoning, subtracting the base domain.
- `--time-measure`: print preparation and solve times.
- `--time-limit SECONDS`: stop reasoning after the given number of seconds.
- `--format human|jsonl`: default `human`.
- `--json-trace`: with JSONL output, emit structured trace JSON instead of formatted string.

Debugging note:

- When investigating a specific TPG/tree, prefer setting `--time-limit` so a potentially looping tree fails fast instead of hanging indefinitely.
- For production-style runs and clean reasoning-speed measurements, do not enable `--time-limit` by default; use it only when an explicit safety bound is part of the task.

JSONL output emits events such as `result`, `variables`, branch-result exceptions, `trace`, optional metrics, reasoner output messages, and exported domain artifacts.

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
- `--debug`: include debug metadata when building the model.
- `--trace`: print expression trace.
- `--verbose`: verbose expression trace.
- `--limit LIMIT`: maximum number of object names; non-negative.
- `--time-measure`: print query execution time.
- `--time-limit SECONDS`: stop query execution after the given number of seconds.
- `--format human|jsonl`: default `human`.

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
