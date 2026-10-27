# Minimal Model Recipe

Read this from [modeling-principles.md](modeling-principles.md) when building a model and it is still unclear how the domain, graph, and situation files fit together. Running, debugging, or explaining an existing model does not need it.

A complete, validated starting point. It was checked with `its_DomainModel 3.0.0-rc.8` (`validate-dsm`, `validate-domain-loqi`) and `its_Reasoner 3.0.0-rc.9` (`reason`, `expression-query`). For other versions treat it as orientation and validate it with the target runtime.

Files in [assets/minimal-model/](../assets/minimal-model/):

```text
minimal-model/
├── model/                      # DomainSolvingModel directory (non-recursive, files recognized by name)
│   ├── domain.loqi             # classes, enums, properties, relationships
│   └── main.tpg                # default (unnamed) graph; `<name>.tpg` would be graph "<name>"
└── situations/                 # specific domains: one task each, kept outside the model directory
    ├── plus_first.loqi
    ├── mul_first.loqi
    └── already_evaluated.loqi
```

## What It Models

Verification of one step in "order of expression evaluation": the student proposes to evaluate operator `X`; the graph checks the proposal against domain constraints instead of comparing it with a precomputed answer (see [modeling-principles.md](modeling-principles.md)).

- `domain.loqi`: `enum State`, class `Element` with object property `state`, subclass `Operator` with relationship `hasOperand(Element)`.
- `main.tpg`: input variable `X: Operator`; a Boolean question with a redirected branch (`ask (...) out true else { ... }`); an iteration with SIM:AND over the operands (`cycle and (selector) with Element Op { ... } out true`); error conclusions carrying `id`, `exception`, `exceptionName`; a correct conclusion carrying `skill`.
- Situations declare objects and bind graph input variables with `var X = obj plus : Operator { ... }` (or `var X = plus` after a separate object declaration).

## Expected Results

| Situation | Result | Exception (`id` / `exceptionName`) |
|---|---|---|
| `plus_first` (`a + b * c`, student picks `+`) | `ERROR` | `operand_not_evaluated` / `OperandNotEvaluated` |
| `mul_first` (student picks `*`) | `CORRECT` | none |
| `already_evaluated` (`+` picked again) | `ERROR` | `already_evaluated` / `AlreadyEvaluated` |

Human trace for `plus_first` shows how the iteration reports each operand:

```text
Result: ERROR
Trace:
  1. QuestionNode => true
  2. CycleAggregationNode => ERROR
     branch[0]: object a
        1. QuestionNode => true
        2. BranchResultNode => CORRECT
     branch[1]: object mul
        1. QuestionNode => false
        2. BranchResultNode [id=operand_not_evaluated, exceptionName=OperandNotEvaluated] => ERROR
```

The reported final node here is the `CycleAggregationNode`, not the inner `BranchResultNode`: the specific error comes from the exceptions list. An expression query over the same situation, `find Operator op { $op.state == State:unevaluated and not forAny Element e { $op=>hasOperand($e) and $e.state != State:evaluated } }`, returns `mul`.

## Running It

Use the execution tool selected in [SKILL.md](../SKILL.md): with MCP, pass the model directory and situation in the form the tool schema declares (`domain__validate-dsm`, `reasoner__reason`); with CLI, see [cli-guide.md](cli-guide.md):

```bash
java -jar "$DOMAIN_CLI_JAR" validate-dsm assets/minimal-model/model
java -jar "$DOMAIN_CLI_JAR" validate-domain-loqi assets/minimal-model/situations/plus_first.loqi assets/minimal-model/model
java -jar "$REASONER_CLI_JAR" reason assets/minimal-model/model assets/minimal-model/situations/plus_first.loqi
```

## Adapting It

Copy the files into the user's project instead of editing the bundled assets. Then grow the model in small steps and validate after each step: domain first (`validate-dsm`), then each situation (`validate-domain-loqi`), then one representative situation per expected outcome (`reason`).

Pitfalls this example already avoids:

- A branch body is always a braced thought branch: `value -> { ... };` or `value -> out;`. There is no `else ->` branch; `else` exists only in the redirected form `ask (boolExp) out true else { ... }` / `var V: T = exp out true else { ... }`.
- Every thought branch must end in `conclude: ...` (or reach one through `out`/`merge`).
- Tree variables are bare names (`X`, `Op`, PascalCase by convention); context variables introduced inside expressions use `$` (`$Op`, `$e`).
- `find T t { ... }` fails at runtime when zero or several objects match; use an iteration or `forAny`/`forAll` when several are possible.
- Exception metadata (`exception = true; exceptionName = "..."`) is read only from conclusion nodes; give conclusions an `id` so traces and exceptions can be correlated.
- `debug:*` calls make `validate-dsm` fail unless its debug option is set; remove them before delivering a model.

For constructs not shown here (tuple questions, `agg`, `while`, fragments, lambdas, `merge`, subcalls), use the grammar and [loqi-tpg-reference.md](loqi-tpg-reference.md).
