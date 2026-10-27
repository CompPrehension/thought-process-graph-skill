# Debugging TPG And Expressions

Use this guide when reasoning gives an unexpected result, an expression evaluates incorrectly, or execution fails with an exception. For simply running a model and reporting its answer, [model-execution.md](model-execution.md) is enough.

The steps work with either execution route chosen in [SKILL.md](../SKILL.md): the CompPrehension MCP tools or the CLI. If the user handed you a trace or limited what you may run or edit, work from what you have and say which steps you skipped.

## Strategy Overview

1. Read the full trace of the failing run.
2. If execution throws, rerun in debug mode to get the partial trace up to the failure.
3. If the trace does not show the needed value, add temporary `debug:*` procedures to a copy of the graph.
4. If behavior is still unclear, read the implementation of the relevant node or operator.

## MCP And CLI Equivalents

| Need | MCP (`reasoner__reason`, `reasoner__expression-query`) | CLI (`reasoner-cli`) |
|---|---|---|
| Full decision-tree trace | always returned in `trace`; `verbose: true` adds node details | default human output; `--verbose` |
| Structured trace | `jsonTrace: true` (default) | `--format jsonl --json-trace` |
| Partial trace on exception, `line` metadata | `debug: true`; the error data carries `partialTrace` / `partialExpressionTrace` | `--debug` |
| Expression trace | `expression-query` with `trace: true` (`debug: true` for partial trace on failure) | `expression-query --trace` (`--debug` for partial trace) |
| Bound a possibly looping run | `timeLimitSeconds` | `--time-limit` |
| Rule out the evaluation cache | `evaluationCache: false` | `--no-evaluation-cache` (`3.0.0-rc.9`+) |
| Validate a model that contains `debug:*` calls | `domain__validate-dsm` with `debug: true` | `validate-dsm --debug` |
| See output of `debug:print`, `debug:dump`, `debug:trace` | not returned unless the tool schema exposes it (the server writes it to its own console) | printed inline; JSONL `reasoner-output` events |

Parameter names come from the server's current schema; if a name above is missing, use the schema's equivalent. Because print-style debug output usually does not reach an MCP client, prefer `debug:breakpoint` / `debug:assert` plus the partial trace there, or switch to the CLI when it is available and print-style output is really needed.

## Step 1 — Read The Trace

CLI forms:

```bash
java -jar "$REASONER_CLI_JAR" reason <MODEL_DIR> <DOMAIN_LOQI> --verbose
java -jar "$REASONER_CLI_JAR" reason <MODEL_DIR> <DOMAIN_LOQI> --format jsonl --json-trace --verbose
java -jar "$REASONER_CLI_JAR" expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose
```

With MCP, call the same operation with `verbose: true` (and `trace: true` for expression queries). [trace-analysis.md](trace-analysis.md) explains how to read the trace and map it to TPG/XML nodes. A mismatch between the expected and actual answer of one `QuestionNode` usually localizes the problem.

When investigating a graph that might loop, set a time limit so the run fails fast. Leave it off for clean performance measurements.

## Step 2 — Partial Trace On Exception

Debug mode does two things at once:

- builds the model with `line` metadata on every node (source line in the `.tpg`);
- collects the trace while reasoning, so if a `ReasoningException` is thrown (including `debug:breakpoint` and failed `debug:assert`), the trace up to that point is reported.

Where it appears:

- CLI human output: `Partial decision tree trace:` with `Failed at: <NodeType> [line=N]`, then the error and stack trace.
- CLI JSONL: `partial-trace` / `partial-expression-trace` events before the `error` event (`reasoner-output` events, if any, come first).
- MCP: fields `partialTrace` / `partialExpressionTrace` in the error data, next to `exceptionName`, `rootCauseMessage`, `stackTrace`, and `variables`.

For expression queries the partial expression trace needs both debug mode and expression tracing. In the CLI, `--no-trace` turns off partial trace collection even with `--debug`.

## Step 3 — Temporary Debug Procedures

Edit a copy of the graph (or the user's graph, if editing it is part of the task) and remove the calls afterwards: models with `debug:*` calls fail production validation (`validate-dsm` without its debug option), including calls nested in expressions, `with (...)` actions, and `conclude: subcall(...)` arguments.

A call has the same form as a graph statement and inside an expression. Inside expressions (including `{ ...; ... }` blocks) only procedures with a return type are allowed: `debug:print`, `debug:trace`, `debug:dump`, `debug:breakpoint`. `debug:assert` (like `eval`) has no return type and is statement-only; otherwise validation fails with `Procedure with no return type can't be used as expression`.

Example (inside a graph with input `X: Operator`, relationship `hasOperand`, and enum `State`):

```loqi
cycle and (X=>hasOperand($Op)) with Element Op {
    debug:print(Op.state);                                   // statement
    ask (debug:trace(Op.state == State:evaluated)) out true else {   // inside an expression
        debug:breakpoint("operand not evaluated");
        conclude: error;
    };
    conclude: correct;
} out true;
```

With the CLI and `--debug`, this prints each operand state, then stops at the breakpoint and reports `Failed at: ProcedureCallNode [line=N]` with the partial trace and current variables.

| Procedure | Effect | Returns |
|---|---|---|
| `debug:print(value)` | prints the value's string form | the value |
| `debug:trace(expr)` | evaluates the unevaluated expression with full expression trace and prints the trace | the value |
| `debug:dump("comment")` | prints `<DEBUG>: comment` and all tree variables (plus scope variables inside expressions) | previous scope value |
| `debug:breakpoint("comment")` | throws `ReasonerBreakpointException`; with debug mode the partial trace ends here | — |
| `debug:assert(cond, "message")` | throws `ReasoningException` when `cond` is false; statement-only | — |

Messages do not interpolate variables. Definitions are in `its_DomainModel` (`.../procedures/Debug*Def.kt`, `AssertPointDef.kt`), implementations in `its_Reasoner` (`its/reasoner/procedures/`).

## Known Behavior Changes And Library Switches

- **SIM:AND semantics** (its_Reasoner `3.0.0-rc.7`+): `NULL` if any branch is `NULL` (or there are no branches), otherwise `ERROR` if any branch is `ERROR`, otherwise `CORRECT`. Older versions ignored `NULL` branches (`CORRECT` + `NULL` gave `CORRECT`). Differences on AND nodes with null-capable branches when comparing against older versions are expected, not a regression.
- **Evaluation cache** (its_Reasoner `3.0.0-rc.2`+): while a read-only expression is evaluated, `ReadOnlyEvaluationCache` indexes relationship links and memoizes search subexpressions. It is on by default. Collecting an expression trace (`debug:trace`, debug mode, expression-query tracing) turns off only the memoization so the trace stays complete; the relationship indexes are still used. If a result looks suspicious, rerun the same input with the cache disabled (table above; `ReasoningOptions(useEvaluationCache = false)` in library code) and compare.

## Step 4 — Read The Source When Behavior Is Unclear

If the trace and debug procedures still do not explain a node or expression, read its implementation in `its_Reasoner` (locate sources via [source-discovery.md](source-discovery.md) if they are not in the workspace):

- Node evaluation: `its/reasoner/nodes/` — `DecisionTreeReasoner.kt` and node-specific files.
- Operator evaluation: `its/reasoner/operators/` — `DomainInterpreterReasoner.kt` and operator-specific files.
- Procedures: `its/reasoner/procedures/`.
- Trace formatting: `its/reasoner/utils/TraceUtils.kt`, `DecisionTreeTraceJsonSerializer.kt`.

When a failure looks like a project defect rather than a modeling mistake, report the runtime version, the exact call or command, the full error, and a minimal reproducer.
