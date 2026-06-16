# Debugging TPG And Expressions

Use this guide when a task asks to debug a thought process graph, a decision tree, or an expression — for example, when reasoning gives an unexpected result, an expression evaluates incorrectly, or the CLI crashes with a Java exception.

## Strategy Overview

1. Run the CLI with `--debug` to get a partial trace on crash.
2. Read the trace (human or JSONL) to find the failing or misbehaving node.
3. Add `debug:*` procedures to the tree/expression to narrow down the value at a specific point.
4. When built-in diagnostics are not enough, read the source code for the relevant operator or node type in `its_Reasoner`.

## Step 1 — Read The Trace First

Before adding debug procedures, always run the reasoning with tracing enabled and read the output:

```powershell
# Human trace — readable tree path
java -jar $REASONER_CLI_JAR reason <MODEL_DIR> <DOMAIN_LOQI> --verbose

# JSONL trace — machine-parseable
java -jar $REASONER_CLI_JAR reason <MODEL_DIR> <DOMAIN_LOQI> --format jsonl --json-trace --verbose
```

For expression queries:

```powershell
java -jar $REASONER_CLI_JAR expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose
java -jar $REASONER_CLI_JAR expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose --format jsonl --json-trace
```

See [trace-analysis.md](trace-analysis.md) for how to read and correlate trace output with TPG/LOQI nodes.

## Step 2 — Use `--debug` For Partial Traces On Exception

When the CLI fails with a Java exception, add `--debug` to get the partial trace that was collected up to the crash point.

For `reason`:

```powershell
java -jar $REASONER_CLI_JAR reason <MODEL_DIR> <DOMAIN_LOQI> --debug --verbose
```

`--debug` activates two things simultaneously:
- **`includeDebugMeta = true`** when building `DomainSolvingModel`: adds `line` metadata to each node based on LOQI source line numbers.
- **Partial trace collection**: if reasoning throws a `ReasoningException`, the CLI prints the partial decision tree trace (or partial expression trace) that was collected before the crash.
  - In human format: printed to `stderr`.
  - In JSONL format: emitted as a `partial-trace` or `partial-expression-trace` event before the error event.

For `expression-query`, `--debug` only prints a partial expression trace if `--trace` is also set:

```powershell
java -jar $REASONER_CLI_JAR expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --debug --trace --verbose
```

Without `--trace`, `--debug` only adds debug metadata to the model and has no partial-trace effect on expression-query crashes.

`--no-trace` disables partial trace collection even when `--debug` is set.

## Step 3 — Add Debug Procedures To The Tree

Debug procedures live in the `debug` namespace. They are defined in `its_DomainModel` and implemented in `its_Reasoner`.

Each procedure can be used in two ways — as a **statement** (`callStmt` in `stmts`) or as an **expression** (`callExpr` in `exp`). There is no special wrapper syntax: the call form `debug:name(args)` is the same in both contexts.

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

Throws `ReasoningException` if `condition` is `false`. Use it to encode invariants that must hold at a specific execution point.

```loqi
debug:assert($x.someProperty == someEnum:Value, "expected someEnum:Value");
ask($x.otherProperty) { ... }
```

Source: `its_DomainModel/…/procedures/AssertPointDef.kt`, `its_Reasoner/…/procedures/AssertPointImpl.kt`.

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
