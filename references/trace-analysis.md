# Reasoner Trace Analysis

Use this guide when an agent must inspect `its_Reasoner` decision-tree traces, expression traces, or trace output and correlate the executed path with the original TPG/LOQI graph.

Before relying on class names or CLI behavior, find the current `its_Reasoner` source jar through [source-discovery.md](source-discovery.md). Do not hard-code a Maven version; use the artifact that is current for the workspace or consuming project.

## Source Files To Inspect

- `its/reasoner/nodes/DecisionTreeTrace.kt`
- `its/reasoner/nodes/DecisionTreeReasoner.kt`
- `its/reasoner/operators/ExpressionTrace.kt`
- `its/reasoner/operators/ExpressionQueryManager.kt`
- `its/reasoner/utils/DecisionTreeTraceJsonSerializer.kt`
- `its/reasoner/utils/TraceUtils.kt`
- `its/reasoner/utils/CliJsonEvents.kt`
- `misc/CLI.kt`

Use [source-discovery.md](source-discovery.md) for `.m2` search commands and source-jar inspection examples.

## Output Format Choice

The agent may choose either human output or JSONL output depending on the analysis task.

Use human output when:
- a person needs a readable reasoning path quickly;
- the task is exploratory and the formatted tree is enough;
- visual indentation and concise metadata are more useful than machine parsing.

Use JSONL output when:
- automation, scripts, or exact event parsing are needed;
- nested traces must be traversed programmatically;
- results, variables, exceptions, trace, metrics, and artifacts should be separated by event type.

Useful commands:

```powershell
java -jar $REASONER_CLI_JAR reason <MODEL_DIR> <DOMAIN_LOQI> --verbose
java -jar $REASONER_CLI_JAR reason <MODEL_DIR> <DOMAIN_LOQI> --format jsonl --json-trace --verbose
```

For expression traces:

```powershell
java -jar $REASONER_CLI_JAR expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose
java -jar $REASONER_CLI_JAR expression-query <MODEL_DIR> <DOMAIN_LOQI> "<QUERY>" --trace --verbose --format jsonl
```

## Decision Tree Trace Shape

`DecisionTree.solve(situation)` returns a `DecisionTreeTrace`. A trace is one executed branch of the tree and is never empty. Its last element must be an `EndingNode` with a `BranchResult`.

Key fields/properties:
- `branchResult`: final branch result, usually from the last element.
- `finalVariableSnapshot`: decision-tree variables at the resulting element.
- `resultingElement`: normally the last element; if the previous element has the same branch result as the final ending node, the previous element is treated as the real result source. This matters after aggregation nodes followed by result nodes with actions.
- `resultingNode`: node that determined the result.
- `branchResultExceptions()`: recursively scans the trace and nested traces for result nodes marked as exceptions in metadata.

Trace element classes:
- `LinkDecisionTreeTraceElement`: normal `LinkNode` execution, such as question/action/procedure nodes. `nodeResult` is the answer used to choose the next outcome.
- `AggregationDecisionTreeTraceElement`: `BranchAggregationNode` or `CycleAggregationNode`; contains `branchTraceMap`, and `nestedTraces()` returns the nested branch traces.
- `WhileCycleDecisionTreeTraceElement`: `WhileCycleNode`; contains `branchTraceList`, one nested trace per iteration.
- `BranchResultDecisionTreeTraceElement`: terminal `BranchResultNode`.
- `RedirectedBranchResultDecisionTreeTraceElement`: terminal result reached through a subinterpreter call; contains `subinterpreterTrace`.

When matching a trace to TPG/LOQI, walk the elements in order and compare node type, metadata `id`, branch/outcome values, and expression text if needed. For aggregations, descend into nested branch traces; for while loops, descend into iteration traces; for redirected results, descend into the redirected subinterpreter trace.

## Human Trace Reading

Human trace formatting is implemented in `TraceUtils.kt`.

The formatted tree trace includes:
- result;
- final variables;
- ordered trace steps;
- nested `branch[...]`, `iter[...]`, and `redirected:` sections;
- node class names and selected metadata.

Important metadata surfaced in human trace headlines:
- `id`
- `line`
- `skill`
- `exceptionName` when `exception=true`
- otherwise raw `exception` if present

With `--verbose`, single-expression nodes without `id` may include LOQI written by `OperatorLoqiWriter`. This helps correlate a trace step with a TPG/LOQI expression when ID metadata is missing.

## JSONL Trace Events

Reasoner JSONL emits separate events:
- `result`: `name = "branchResult"`, `value = trace.branchResult`.
- `variables`: `value` is `finalVariableSnapshot`.
- `exceptions`: `found` plus `value[]` entries with `result`, `exceptionName`, and `id`.
- `trace`: if `--json-trace` is set, `value` is structured JSON; otherwise it is formatted text.
- `reasoner-output`: debug sink messages (`"level": "debug"`), including output from `debug:*` TPG procedures.
- `metric`: optional timing events.
- `artifact` or `service`: optional exported domain output.

Partial-trace events (emitted when `--debug` is set and a `ReasoningException` is thrown):
- `partial-trace`: partial decision tree trace collected up to the crash point; structured JSON with `--json-trace`, formatted text otherwise.
- `partial-expression-trace`: partial expression trace; emitted for `reason --debug` when the exception carries an expression trace, or for `expression-query --debug --trace`.

For `expression-query` JSONL:
- `expression-query-result`: `objects` list of matching object names.
- `expression-trace`: expression trace; structured JSON with `--json-trace`, formatted text otherwise.

Structured trace JSON has:
- root `branchResult`, `finalVariables`, `elements`.
- each element: `nodeType`, `nodeId`, `nodeResult`, `variables`.
- with `--verbose`, each element also has `node`, usually `node.toString()`.
- aggregation elements add `branches[] = { branch, trace }`.
- while elements add `iterations[] = { index, trace }`.
- redirected result elements add `redirectedTrace`.

`nodeId` comes from `node.metadata.getString("id")`. If it is absent, switch to human verbose or JSON trace `node` text and compare expressions/position.

## Exception Metadata

Exception detection is metadata-driven and only applies to `BranchResultNode`.

The reasoner treats a result node as an exception when:
- metadata `exception` is string `true` ignoring case, or `1`.

The exception event then uses:
- `exceptionName`: metadata `exceptionName`, defaulting to `unknown`.
- `id`: metadata `id`, defaulting to `unknown`.
- `result`: the `BranchResultNode.value`.

So if a final node "unexpectedly" appears as an exception, inspect the tree metadata on that `BranchResultNode`, especially `id`, `exception`, `exceptionName`, `skill`, and `line`.

## Expression Trace Shape

`ExpressionQueryManager.query(..., collectTrace = true)` returns `ExpressionQueryResult`:
- `objectRefs`: extracted object refs from the evaluated value.
- `val`: raw evaluated value.
- `trace`: list of root `ExpressionTrace` nodes.

`ExpressionTrace` fields:
- `expr`: the evaluated `Operator`.
- `val`: evaluated value.
- `isValueAnnotated`: whether the value should be displayed.
- `children`: nested expression evaluations.

CLI `expression-query --trace --format jsonl` emits an `expression-trace` event. With `--json-trace`, `value` is a structured JSON tree (list of `ExpressionTrace` as JSON objects); without it, `value` is a formatted string. Use `--verbose` to include operator class names plus LOQI written by `OperatorLoqiWriter`. Correlate the expression trace to TPG/LOQI by comparing the written expression text with the expression inside the relevant XML wrapper or LOQI statement.

## Analysis Checklist

1. Choose human or JSONL output based on whether the trace is being read manually or parsed programmatically.
2. Read the result, final variables, exception summary, and trace body.
3. Follow nested traces recursively before deciding which node determined the result.
4. Match `nodeId` to XML `_id` or LOQI metadata `id`.
5. If there is no ID, match by node type, outcome value, expression text, and traversal position.
6. If an exception was reported, inspect only `BranchResultNode` metadata: `exception`, `exceptionName`, and `id`.
7. If expression behavior is unclear, run `expression-query --trace --verbose` and compare written LOQI expressions to the XML expression wrappers or original TPG/LOQI.
