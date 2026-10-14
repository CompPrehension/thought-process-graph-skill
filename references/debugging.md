# Debugging TPG And Expressions

Use this guide when a task asks to debug a thought process graph, a decision tree, or an expression — for example, when reasoning gives an unexpected result, an expression evaluates incorrectly, or the CLI crashes with a Java exception.

## Strategy Overview

1. Run the CLI with `--debug` to get a partial trace on crash.
2. Read the trace (human or JSONL) to find the failing or misbehaving node.
3. Add `debug:*` procedures to the tree/expression to narrow down the value at a specific point.
4. When built-in diagnostics are not enough, read the source code for the relevant operator or node type in `its_Reasoner`.

## Step 1 — Read The Trace First

Before adding debug procedures, always run the reasoning with tracing enabled and read the output:

```bash
# Human trace — readable tree path
java -jar "$REASONER_CLI_JAR" reason <MODEL_DIR> <DOMAIN_LOQI> --verbose

# JSONL trace — machine-parseable
java -jar "$REASONER_CLI_JAR" reason <MODEL_DIR> <DOMAIN_LOQI> --format jsonl --json-trace --verbose
```

For expression queries:

```bash
java -jar "$REASONER_CLI_JAR" expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose
java -jar "$REASONER_CLI_JAR" expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose --format jsonl --json-trace
```

See [trace-analysis.md](trace-analysis.md) for how to read and correlate trace output with TPG/LOQI nodes.

## Step 2 — Use `--debug` For Partial Traces On Exception

When the CLI fails with a Java exception, add `--debug` to get the partial trace that was collected up to the crash point.

For `reason`:

```bash
java -jar "$REASONER_CLI_JAR" reason <MODEL_DIR> <DOMAIN_LOQI> --debug --verbose
```

`--debug` activates two things simultaneously:
- **`includeDebugMeta = true`** when building `DomainSolvingModel`: adds `line` metadata to each node based on LOQI source line numbers.
- **Partial trace collection**: if reasoning throws a `ReasoningException`, the CLI prints the partial decision tree trace (or partial expression trace) that was collected before the crash.
  - In human format: printed to `stderr`.
  - In JSONL format: emitted as a `partial-trace` or `partial-expression-trace` event before the error event.

For `expression-query`, `--debug` only prints a partial expression trace if `--trace` is also set:

```bash
java -jar "$REASONER_CLI_JAR" expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --debug --trace --verbose
```

Without `--trace`, `--debug` only adds debug metadata to the model and has no partial-trace effect on expression-query crashes.

`--no-trace` disables partial trace collection even when `--debug` is set.

## Step 3 — Add Debug Procedures To The Tree

Debug procedures live in the `debug` namespace. They are defined in `its_DomainModel` and implemented in `its_Reasoner`.

Procedures can be used in two ways — as a **statement** (`callStmt` in `stmts`) or as an **expression** (`callExpr` in `exp`). There is no special wrapper syntax: the call form `debug:name(args)` is the same in both contexts. In expressions (including `{ ...; ... }` blocks, even when the value is unused) only procedures with a return type are valid: `debug:print`, `debug:trace`, `debug:dump`, `debug:breakpoint`. `debug:assert` has no return type and is statement-only; inside an expression validation fails with `Procedure with no return type can't be used as expression`.

The production check (`validate-dsm` without `--debug` rejects `debug:` procedures) covers both statement calls and debug calls inside expressions, including `with (...)` actions and `conclude: subcall(...)` arguments.

As a statement, it appears between semicolons in a TPG branch body:

```loqi
tpg Example(x: SomeClass) {
  debug:dump("start");
  ask($x.someProperty) {
    someEnum:Value -> conclude: correct
    else -> conclude: error
  }
}
```

As an expression, it can wrap any sub-expression inline — the procedure evaluates the expression and returns its value:

```loqi
ask(debug:trace($x.someProperty)) {
  someEnum:Value -> conclude: correct
  else -> conclude: error
}
```

### `debug:trace(expr)`

Evaluates `expr`, collects a full expression trace, and prints it to CLI output. Returns the evaluated value. Use this to see which sub-expressions were evaluated and what they returned.

The argument has type `ExpressionType` — the expression is passed unevaluated and then evaluated internally with trace collection.

```loqi
// Inline — wraps the expression being asked about
ask(debug:trace($x.someProperty)) { ... }

// Or as a statement before another node
debug:trace($x.someProperty);
ask($x.someProperty) { ... }
```

The trace is formatted as indented text with each sub-expression and its value. Output appears inline in human mode or as `reasoner-output` events (level `debug`) in JSONL mode.

Source: `its_DomainModel/…/procedures/DebugTraceDef.kt`, `its_Reasoner/…/procedures/DebugTraceImpl.kt`.

### `debug:print(obj)`

Prints the string representation of any value to CLI output. Returns the value unchanged. Use it to check the current value of a variable or property without stopping execution.

```loqi
debug:print($currentVar);
debug:print($x.someProperty)
```

Source: `its_DomainModel/…/procedures/DebugPointDef.kt`, `its_Reasoner/…/procedures/DebugPointImpl.kt`.

### `debug:dump("comment")`

Prints all current tree-level variables and scope variables to CLI output, prefixed with `<DEBUG>: comment`. Returns the previous scope value (transparent when used as a statement). Use it when you need a full variable snapshot at a specific execution point.

```loqi
debug:dump("before aggregation");
agg and { ... }
```

Source: `its_DomainModel/…/procedures/DebugDumpPointDef.kt`, `its_Reasoner/…/procedures/DebugDumpPointImpl.kt`.

### `debug:breakpoint("comment")`

Throws `ReasonerBreakpointException` with the given comment. Stops execution immediately. Use it to confirm which branch is reached, or to force a crash at a known point so the partial trace ends there.

```loqi
someEnum:Value -> {
  debug:breakpoint("reached unexpected branch");
  conclude: error
}
```

Combine with `--debug` on the CLI to see the partial trace collected up to this point.

Source: `its_DomainModel/…/procedures/DebugBreakpointDef.kt`, `its_Reasoner/…/procedures/DebugBreakpointImpl.kt`.

### `debug:assert(condition, "message")`

Throws `ReasoningException` if `condition` is `false`. Use it to encode invariants that must hold at a specific execution point. Statement-only: it cannot appear inside an expression or block (see above).

```loqi
debug:assert($x.someProperty == someEnum:Value, "expected someEnum:Value");
ask($x.otherProperty) { ... }
```

Source: `its_DomainModel/…/procedures/AssertPointDef.kt`, `its_Reasoner/…/procedures/AssertPointImpl.kt`.

## Known Behavior Changes And Library Switches

- **SIM:AND semantics** (its_Reasoner `3.0.0-rc.7`+): `NULL` if any branch is `NULL` (or there are no branches), otherwise `ERROR` if any branch is `ERROR`, otherwise `CORRECT`. Older versions ignored `NULL` branches (`CORRECT` + `NULL` gave `CORRECT`). When comparing traces with results from older versions, differences on AND nodes with null-capable branches are expected, not a regression.
- **Evaluation cache** (its_Reasoner `3.0.0-rc.2`+): while a read-only expression is evaluated, `ReadOnlyEvaluationCache` indexes relationship links and memoizes search subexpressions. It is on by default (`ReasoningOptions.useEvaluationCache = true`). Collecting an expression trace (`debug:trace`, `reason --debug`, `expression-query --trace`) turns off only the memoization so the trace stays complete; the relationship indexes are still used. To rule the cache out when a result looks suspicious, rerun the same input without it and compare: CLI `--no-evaluation-cache` on `reason` / `expression-query` (its_Reasoner `3.0.0-rc.9`+), MCP `reasoner__reason` / `reasoner__expression-query` with `evaluationCache: false`, or `ReasoningOptions(useEvaluationCache = false)` in library code.

## Step 4 — Read The Source When Behavior Is Unclear

If trace output and debug procedures still do not explain why a node or expression behaves unexpectedly, read the reasoner source directly. Most operators and node types are straightforward once you read the implementation.

Relevant source locations in `its_Reasoner`:

- Node evaluation: `its/reasoner/nodes/` — `DecisionTreeReasoner.kt` and node-specific files.
- Operator evaluation: `its/reasoner/operators/` — `DomainInterpreterReasoner.kt` and operator-specific files.
- Procedures: `its/reasoner/procedures/` — one file per procedure.
- Trace formatting: `its/reasoner/utils/TraceUtils.kt`, `DecisionTreeTraceJsonSerializer.kt`.

Use [source-discovery.md](source-discovery.md) to locate current source jars if `its_Reasoner` sources are not in the local workspace.

## JSONL Debug Event Types

When `--format jsonl` is active, debug-related events have the following `type` values:

| Event type | When emitted |
|---|---|
| `partial-trace` | `reason --debug` on exception; structured JSON with `--json-trace`, formatted text otherwise |
| `partial-expression-trace` | `reason --debug` or `expression-query --debug --trace` on exception; same JSON/text distinction |
| `reasoner-output` | Messages from `ReasonerOutput.println`, including debug procedure output; always `"level": "debug"` |

These appear before the final error event. When parsing JSONL output from a crashed run, look for these event types first before the error line.
