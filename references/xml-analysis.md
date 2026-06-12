# Compiled XML Tree Analysis

Use this guide when an agent must inspect compiled XML thought-process trees/graphs and correlate them with a TPG/LOQI file or reasoner trace.

Before relying on class names or XML behavior, find the current `its_DomainModel` source jar through [source-discovery.md](source-discovery.md). Do not hard-code a Maven version; use the artifact that is current for the workspace or consuming project.

## Source Files To Inspect

- `its/model/nodes/xml/DecisionTreeXMLBuilder.kt`
- `its/model/nodes/xml/DecisionTreeXMLWriter.kt`
- `its/model/nodes/DecisionTree.kt`
- `its/model/nodes/DecisionTreeElement.kt`
- `its/model/nodes/DecisionTreeNode.kt`
- `its/model/nodes/ThoughtBranch.kt`
- `its/model/nodes/LinkNode.kt`
- `its/model/nodes/Outcomes.kt`
- node classes such as `QuestionNode`, `TupleQuestionNode`, `FindActionNode`, `BranchAggregationNode`, `CycleAggregationNode`, `WhileCycleNode`, `ProcedureCallNode`, `BranchResultNode`, `BranchResultRedirectingNode`.

Use [source-discovery.md](source-discovery.md) for `.m2` search commands and source-jar inspection examples.

## Builder And Writer Classes

Compiled XML is parsed by `DecisionTreeXMLBuilder` and written by `DecisionTreeXMLWriter`.

Important builder/writer details:
- `DecisionTreeXMLBuilder` builds a full `DecisionTree` from `DecisionTree` or legacy `StartNode` root tags.
- `DecisionTreeNodeXMLBuilder` builds individual `DecisionTreeNode` tags.
- `DecisionTreeXMLWriter.SHOULD_USE_CDATA_EXPRESSIONS` controls whether expressions are written as LOQI CDATA or as expression XML.
- `DecisionTreeXMLBuilder.collectMetadata()` reads metadata attributes from XML.
- `DecisionTreeXMLWriter.withMetadataOf()` writes metadata back to XML.

When XML behavior is unclear, these two classes are the authority for tag names, attributes, metadata, and compatibility aliases.

## Root Structure

```xml
<DecisionTree>
  <InputVariables>
    <DecisionTreeVarDecl name="..." type="..."/>
    <AdditionalVarDecl name="..." type="...">
      <Expression>...</Expression>
    </AdditionalVarDecl>
  </InputVariables>
  <ThoughtBranch>...</ThoughtBranch>
</DecisionTree>
```

`DecisionTree` contains:
- `variables`: input `DecisionTreeVarDecl` values.
- `implicitVariables`: `AdditionalVarDecl` assignments evaluated before the main branch if missing.
- `mainBranch`: one `ThoughtBranch`.

A `ThoughtBranch` contains exactly one start node. Traversal then follows `Outcome` edges until a result node is reached.

## Node Tags

Common node tags:
- `QuestionNode`: has `Expression`, optional `Triviality`, optional `isSwitch="true"`, and `Outcome` children.
- `TupleQuestionNode`: has `Part` children with expressions and possible part outcomes, plus tuple-valued `Outcome` children.
- `FindActionNode`: has `DecisionTreeVarDecl`, main `Expression`, optional `FindError`, optional `AdditionalVarDecl`, and boolean outcomes.
- `ProcedureCallNode`: has `procedure` class attribute, `ExpressionArguments`, and boolean outcomes.
- `BranchAggregationNode`: has `operator`, multiple `ThoughtBranch` children, and `BranchResult` outcomes.
- `CycleAggregationNode`: has `operator`, `DecisionTreeVarDecl`, `SelectorExpression`, optional `FindError`, one `ThoughtBranch`, and `BranchResult` outcomes.
- `WhileCycleNode`: has `SelectorExpression`, one `ThoughtBranch`, and `BranchResult` outcomes.
- `BranchResultNode`: has `value` and optional `Expression`.
- `BranchResultRedirectingNode`: has `procedure`, `ExpressionArguments`, and optional `Expression`.

Compatibility aliases accepted by the builder:
- root `StartNode` can build a `DecisionTree`.
- `LogicAggregationNode` can build a `BranchAggregationNode`.
- `WhileAggregationNode` can build a `WhileCycleNode`.

## Outcome Edges

Outcome edges are represented as:

```xml
<Outcome value="...">
  <NextNodeTag .../>
</Outcome>
```

The XML builder parses outcome values by type:
- booleans, numbers, strings;
- enum values like `EnumName:ValueName`;
- classes like `class ClassName`;
- objects like `obj ObjectName`;
- branch results like `CORRECT`, `ERROR`, `NULL`;
- tuple values like `(a; b; *)`, where `*` means wildcard inside tuples.

The reasoner chooses the next node by comparing the current node's result/answer to these outcome values.

## Expressions

Expression-bearing XML wrappers include:
- `Expression`
- `SelectorExpression`
- `Triviality`
- expressions under `ExpressionArguments`

The builder accepts either:
- CDATA/text LOQI expressions parsed by `OperatorLoqiBuilder.buildExp`;
- nested expression XML parsed by `ExpressionXMLBuilder`.

When correlating XML with LOQI or expression traces, compare the expression under the relevant wrapper with the LOQI produced by `OperatorLoqiWriter` in verbose traces.

## Metadata Encoding

All decision-tree elements inherit `DecisionTreeElement.metadata`.

XML metadata is encoded as attributes whose names start with `_`:
- unlocalized metadata: `_id="node-1"`, `_exception="true"`, `_exceptionName="..."`.
- localized metadata: `_ru_label="..."` means loc code `ru`, property `label`.

`DecisionTreeXMLBuilder.collectMetadata()` reads these attributes into `MetaData`. `DecisionTreeXMLWriter.withMetadataOf()` writes metadata back with the same `_` prefix convention.

Metadata can exist on nodes, `ThoughtBranch`, `Outcome`, tuple parts, tuple part outcomes, and find-error categories. When searching for a reasoner trace node, first search for `_id="<nodeId>"`, then widen to related edge or branch metadata.

## Trace-To-XML Matching

For trace-to-XML matching:
1. Start with trace `nodeId`; search XML for `_id="<nodeId>"`.
2. If no ID exists, compare trace `nodeType` to XML tag name.
3. Compare trace `nodeResult` with the surrounding `Outcome value`.
4. Compare expression text under `Expression`, `SelectorExpression`, or `Triviality`.
5. Use traversal position inside `ThoughtBranch`, aggregation branch, while iteration, or redirect.
6. For exceptions, inspect `BranchResultNode` metadata: `_exception`, `_exceptionName`, `_id`.

If XML and LOQI disagree, re-run LOQI-to-XML conversion and inspect `DecisionTreeXMLBuilder`/`DecisionTreeXMLWriter` from the current `.m2` source jar before assuming the XML shape by memory.

## Analysis Checklist

1. Identify whether the XML came from `tree-loqi-to-xml` and whether expressions are CDATA LOQI or nested expression XML.
2. Locate root `DecisionTree`, `InputVariables`, and the first `ThoughtBranch`.
3. Follow `Outcome` edges recursively, respecting aggregation and cycle nodes.
4. Search metadata attributes with `_` prefixes, especially `_id`.
5. Map XML node tags to model classes from `its/model/nodes`.
6. If matching to a trace, use `trace-analysis.md` for trace shape and output-format choices.
