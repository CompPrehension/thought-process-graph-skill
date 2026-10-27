# Apply An Existing Model

Use this route to obtain and interpret answers from a supplied TPG or expression query. A graph can serve as a reasoning tool for the user's task without any graph development or debugging.

## Establish The Operation

- **`reason`** applies an existing graph to a concrete situation: its conclusion, computed variables, and whatever explanation the returned trace supports.
- **`expression-query`** queries domain objects through a LOQI expression.

Identify the model directory, the situation, the graph name (empty for the default `main.tpg` / `tree.xml`), an optional tag, and the input variables the graph expects (the situation binds them, e.g. `var X = obj1`). If a required input is missing, ask for it rather than inventing domain facts or silently choosing another graph. Passing complete supplied inputs needs neither the grammar nor the modeling guide; writing a new situation or expression is LOQI work.

## Call The Tool

With MCP, the tool description and schema are the entry point: prepare the inputs and call it directly. Use the file/directory payload format the schema declares; a path on the client machine is not necessarily readable by a remote server, and inline contents avoid that problem when supported. Use whatever version information the server returns; a missing version is not a reason to search Maven or sources first. A successful call shows that this runtime accepted these inputs, not a general compatibility claim.

With the CLI, use the `reason` or `expression-query` section of [cli-guide.md](cli-guide.md), locating only the jar that operation needs.

Request only the output the task needs:

- For an ordinary result, the conclusion and returned values are enough (CLI `reason --no-trace` prints just those).
- For automation, parse MCP's structured result or CLI JSONL events by field/type rather than scraping text.
- To explain the reasoning path, read the returned trace with [trace-analysis.md](trace-analysis.md), only as deep as the explanation needs.

A run computes changes to the situation in memory; the input files stay as they are. Export the resulting domain only when the user wants it.

## Interpret The Answer

| Returned information | How to use it |
|---|---|
| Branch result (`branchResult`, or the CLI JSONL `result` event) | Report the graph's conclusion. Interpret result names using the model's meaning; not every graph is a Boolean correctness checker. |
| Variables / final values | Extract the values the user needs, keeping object names and types. |
| Exceptions (`exceptions`: `id`, `exceptionName`, `result`) | The specific modeled error, e.g. a violated rule in the student's action. Distinguish it from a failure to execute the tool. |
| Final node | The node that determined the result. After an aggregation or iteration it is often the aggregation node itself; the concrete error is then in the exceptions and the trace. |
| Trace | Use it to explain the conclusion when asked. |
| Query objects (`objects`) | Report the matching objects. With a result limit, the list may not be exhaustive. |
| Execution error or incomplete result | Say that execution failed and report the actual diagnostic instead of inventing a conclusion. |

Keep claims tied to the model and situation used: a run shows what this model concludes for these inputs, not that its domain rules are right or its design is sound. If the response lacks something needed for the explanation, say what is missing instead of guessing the path.

## When To Stop

These stopping rules apply when getting the answer is the whole task. If the run is one step of a larger task (checking a graph for errors, authoring, debugging), return the result to that task: a failure or a surprising answer there is a finding to analyze, not a place to stop.

Return the decision, values, or matches, with the explanation asked for. Running a graph does not by itself call for validating its design, reading the papers, or investigating the implementation.

If the call fails because of an invocation mistake (wrong payload format, graph name, tag), fix it and retry. If the user wants to know why the model behaves unexpectedly, that is the `debug` route; otherwise report the failure or the surprising answer accurately and leave the model unchanged.
