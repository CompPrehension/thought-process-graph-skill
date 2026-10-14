# LOQI And Thought Process Graph Reference

Use this as a compact orientation only. For exact syntax, read the latest `LoqiGrammar.g4` from local/current sources through [source-discovery.md](source-discovery.md):

- Workspace source: `its_DomainModel/src/main/antlr4/its/model/definition/loqi/LoqiGrammar.g4`
- Source jar path inside current artifacts: `its/model/definition/loqi/LoqiGrammar.g4`

## Top-Level LOQI Domain Declarations

The domain model accepts one or more declarations:

- `class`: class declaration, optional parent, properties, relationships, and metadata.
- `enum`: enum declaration and enum values.
- object declaration: `obj name : Class { ... }` or `name : Class { ... }`.
- `meta for ... [ ... ]`: metadata extension.
- `values for class ClassName { ... }`: class-level property values.
- `var a, b = name`: variable declaration form.

Relationship declarations support relationship kinds:

- scale: `linear`, `partial`
- quantifier: `{min->max}`, where max can be `*`
- dependency: `opposite to`, `transitive to`, `between to`, `closer to`, `further to`

## Expressions

Important expression forms include:

- literals: integers, doubles, booleans, strings, enum values.
- object/class literals: `obj:Name`, `class:Name`.
- variable reference: `$name`.
- calls: `namespace:name(args...)`.
- relationship/property access: `expr->rel`, `expr.prop`, `expr.class()`.
- relationship checks: `expr => rel(args...)`.
- quantifiers: `forAny`, `forAll`.
- search: `find`, `findExtreme`.
- mutation-like forms: `+ obj:Class(...)`, `+=>`, `-=>`, assignment.
- blocks and conditionals: `{ ... }`, `if (...) ... else ...`, ternary `? :`.

Expression precedence is grammar-defined. If changing expression syntax, also inspect writer/parser code such as `OperatorLoqiWriter` and builder code; the grammar comment explicitly warns that expression priority changes must be mirrored there.

Current packaged grammar also includes tree-level lambda helpers:

```loqi
lambda HelperName(arg: Type, other: Type) = someExpression
```

In the current grammar, `lambda` is a sibling of `meta` and `fragment` in `treeDeclHelpers`, so it can appear around a `tpg` declaration the same way other tree helpers do.

## Thought Process Graph Syntax

The tree/graph declaration starts with `tpg`:

```loqi
tpg GraphName(input: Type, other: Type = defaultExp) {
  ...
}
```

The grammar names this `treeDecl`, but the keyword is `tpg` (`THOUGHT_PROCESS_GRAPH`).

Tree variables:

```loqi
name: type
name: type = exp
```

Statements can be:

- `conclude: correct|error|true|false|null`
- `merge`
- `while (exp) ...`
- `agg and|or|mutex|hyp ...`
- `cycle aggregation (exp) with Type item ...`
- `var name: Type = exp ...`
- `ask (...) ...`
- namespace call statements with optional redirected branches.

`merge` is important in newer TPG sources. In current builder logic it means "merge into the continuation of this branch". It is not a standalone terminal action: if there is no following continuation to merge into, the builder throws an error.

Branches use either outcome lists or expression lists:

```loqi
correct -> { conclude: correct };
error -> out;
someExp -> { ... };
```

Supported outcomes:

- `correct`
- `error`
- booleans `true` and `false`
- `null`

Aggregations:

- `and`
- `or`
- `mutex`
- `hyp`

Question forms:

```loqi
ask (exp) { ... }
ask switch (exp) { ... }
ask (exp) with trivial [exp] { ... }
ask tuple (exp; exp) {
  (a; b) -> { ... }
}
```

Fragments are reusable thought branches:

```loqi
fragment Name(args...) {
  ...
}
```

Call a fragment as a statement with the mandatory `fragment` namespace: `fragment:Name(X);`. Backticks around `fragment` are accepted but not needed. A bare `Name(X)` is resolved as a procedure, not a fragment. Arguments must be tree variable names.

A fragment call may carry its own metadata block before `;` (only `[...]`, not `as ID`), also after full redirection branches:

```loqi
fragment:Name(X) [ skill = "division" ; RU.explanation = "..." ; ];
fragment:Name(X) { error -> out; } [ skill = "division" ; ];
```

- Call metadata is added to every `conclude` of the inlined body, including nested fragment calls; conclusions substituted by the call's own redirections keep only their own metadata.
- A key set both on a fragment conclusion and on the call is a build error (`Metadata property ... is set both in fragment ... and at its call`); keys are compared regardless of localization.
- Metadata on a non-fragment procedure call is a build error (`Call metadata is supported only for fragments`).
- Metadata templates inside a fragment (and `meta for` blocks attached to fragment nodes) have formal argument references (`${A}`, `$A`) renamed to the caller's variables per inlined copy; `$$name` context variables are untouched. An unparseable `${...}` is a build error.

Lambdas are reusable expression helpers:

```loqi
lambda Name(arg: Type, other: Type) = expression
```

Metadata helpers can appear around a tree declaration:

```loqi
meta for SomeName [ key = "value" ]
```

A metadata alias (`as ID`, `-[ID]->`, `out[ID]`) is also written into the model as `alias = "ID"`, unless `meta for ID` sets an explicit `alias`. Aliases starting with `_` are service handles and do not produce `alias`; `decompile-tree` / `TreeLoqiWriter` emits them as `_nN` (from `TEMPLATING_ID`) and `_mN` (synthetic) when the element's own `alias` is unusable or already taken by a copy with different metadata.

Procedure calls inside expressions (`callExpr`, including `{ ...; ... }` blocks) must have a return type: `subcall`, `subcall_mut`, `debug:print`, `debug:trace`, `debug:dump`, `debug:breakpoint` are fine; `eval` and `debug:assert` fail validation with `Procedure with no return type can't be used as expression` and may only be used as graph statements.

## Grammar Checks

When asked whether LOQI code is syntactically correct:

1. Find the latest grammar in sources or source jar.
2. Inspect the relevant parser builder (`DomainLoqiBuilder`, `TreeLoqiBuilder`, or `OperatorLoqiBuilder`) if grammar alone is ambiguous.
3. If the syntax touches `fragment`, `lambda`, `merge`, or expression precedence, also inspect the corresponding writer code before assuming a parseable construct is fully supported end-to-end.
4. Prefer CLI validation/conversion over visual inspection:
   - domain files: `domain-cli validate-domain-loqi`
   - model directories: `domain-cli validate-dsm`
   - tree LOQI: `domain-cli tree-loqi-to-xml --model-dir ...`
   - expression snippets: `reasoner-cli expression-query` when the expression must be executable against a domain.
