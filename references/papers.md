# Scientific Papers (Theory, Terminology, Citations)

Two papers are bundled in `assets/`. They are the source for *why* thought process graphs exist, what the node kinds mean pedagogically, and how the approach is positioned against other ITS domain-modeling methods. They are **not** a reference for current syntax or tool behavior: both were written before LOQI2 and the `3.0` toolchain, and the expression examples in them use an older notation. For anything that must parse or run, use `LoqiGrammar.g4`, the CLI, and the wiki (see [docs-workflow.md](docs-workflow.md)).

## When To Open A Paper

Open a paper when the task needs one of these:

- an explanation of the idea for a reader who is not the tool's developer (report, thesis chapter, README intro, presentation);
- a citation or bibliographic entry;
- the pedagogical meaning of a node kind or of a modeling decision (why a verification tree checks constraints in a particular order, why iteration is preferred to storing arrays, etc.);
- the difference between solving/verification/hinting trees or between constraint-based and rule-based domain models;
- historical naming (decision tree, thought process tree, TPT, TPG) to interpret older files, wiki pages, or discussions.

Do not open a paper to answer "is this LOQI valid", "what does this CLI flag do", or "why does the reasoner pick this branch". The papers cannot answer those, and reading 47 pages for that wastes context.

## Which Paper

| Need | Paper |
|---|---|
| Short overview of the approach, the framework parts (editor, interpreter, question generator), development-time and performance figures | KII 2023 (12 pages, Russian, English abstract at the end) |
| Deep methodology: node semantics, dictionaries, worked example (order of expression evaluation), lessons learned, formalization, list of built tutors | BICA 2023 / Cognitive Systems Research preprint (47 pages, English) |
| Russian-language terminology for a Russian text | KII 2023 |
| English-language terminology for an English text | BICA |

If both would do, prefer KII for a quick answer and BICA when the question is about modeling methodology.

## How To Read Them

The PDFs are text-based (not scans). Read only the pages you need; the page maps below are for that.

```bash
# Whole text into a scratch file, then search it
pdftotext -layout assets/bica-2023-educational-models-for-cognition.pdf /tmp/bica.txt
rg -n -i "verification tree|hinting tree|dictionar" /tmp/bica.txt

# A page range only
pdftotext -layout -f 35 -l 38 assets/bica-2023-educational-models-for-cognition.pdf -
```

If a PDF-capable file reader is available (for example, a `Read` tool with a `pages` parameter), reading a page range directly is fine too.

## KII 2023 — `assets/kii-2023-thought-process-graph.pdf`

**Citation.** Крыгин А. И., Гумеров М. Р., Москаленко Н. А., Сычев О. А. Фреймворк для разработки интеллектуальных обучающих систем на основе моделей предметных областей в виде деревьев решений // КИИ-2023 (Конференция по искусственному интеллекту), Волгоградский государственный технический университет, 2023. English title: *A Framework for Developing Intelligent Tutoring Systems Based on Domain Models in the Form of Decision Trees*.

**What it contains** (PDF pages):

| Pages | Content |
|---|---|
| 1–3 | Introduction: why domain models for ITS are expensive; related tools (ASSISTment Builder, Thought Process Language by O'Rourke et al., CTAT figures of 200–300 dev hours per hour of content) |
| 4 | Section 1: node kinds (question, branching, result, action, cycle), dictionaries (classes, properties, relationships), *computation trees* vs *verification trees* and what each enables |
| 5–7 | Section 2: the framework — draw.io editor plugin, first-order expression language over RDF knowledge graphs, Blockly editor, interpreter vs compiling to Apache Jena rules (why Jena rules are hard: no disjunction, no negation over groups, no priorities) |
| 8–9 | Section 3: four ITS cores built; timing plots; 62 nodes / 41 expressions / 74 pedagogical questions for order-of-evaluation; 20 person-hours vs 150 for handwritten Jena rules; Table 1 with node counts and development time for three domains |
| 9 | Conclusion and future work (compile to Jena / Prolog / ASP / SPARQL) |
| 10–11 | References |
| 11–12 | English title and abstract |

Note: the paper's Figure 1 (expression examples) is an image; its text is not extractable.

## BICA 2023 — `assets/bica-2023-educational-models-for-cognition.pdf`

**Citation.** Sychev O. Educational Models for Cognition: Methodology of Modeling Intellectual Skills for Intelligent Tutoring Systems. Preprint submitted to *Cognitive Systems Research* (Elsevier), revision 2, April 2025; based on a BICA 2023 contribution. Before citing formally, check whether the journal version has been published and cite that instead.

**What it contains** (PDF pages):

| Pages | Content |
|---|---|
| 1 | Highlights and abstract |
| 2–5 | Introduction: feedback kinds, why LLM-only error labeling is insufficient for STEM feedback, hybrid symbolic AI argument |
| 6–8 | Related work: structural vs content-related domain models, rule-based vs constraint-based, scaffolding kinds, comparison with Thought Process Language |
| 9–16 | Section 3, *Thought Process Trees*: the six node kinds (question, action, outcome, branching — aggregated and independent, iteration, recursive) with figures; the dictionaries (classes, class properties, object properties, relationships, questions, actions) |
| 17–24 | Section 4.1: full worked example — verification tree for order of expression evaluation, step by step |
| 25–30 | Section 4.2, lessons learned: storing arrays vs iteration; sequence of questions vs independent branches; questions about classes vs class properties |
| 30–34 | Section 4.3: explaining errors for single-answer tasks; task solving vs constraints; order of verifying constraints; comparison of error coverage (23 vs 54 vs more error types) |
| 35–37 | Section 5, formalization: dictionaries as ontologies with static typing, nodes as statements and labels as expressions, the expression language (example expression (1)), executing over RDF, interpreter vs compiling to rules |
| 38–39 | Section 6: the five modeled domains, development time 10–50 person-hours per tree, the six tasks trees are used for (error sets, per-step checking, hints, dialogue, solving and classifying generated problems) |
| 39–40 | Conclusion and future work |
| 41–47 | References |

## Mapping Paper Terms To The Current Implementation

The papers use methodology-level names; the code and wiki use implementation names. Use this table to translate, and confirm details in the wiki page on nodes (`docs/decision_tree/1. Модели (its_DomainModel)/Дерево решений/Узлы.md`) when precision matters.

| Paper term | Current name / LOQI construct |
|---|---|
| Decision tree, Thought Process Tree (TPT) | Thought process graph (TPG), `tpg` declaration; code classes are still named `DecisionTree*` |
| Question node | `QuestionNode`, `ask (...) { ... }` (tuple form: `TupleQuestionNode`, `ask tuple`) |
| Action node (find object, calculate) | `FindActionNode` / tree variable assignment: `var name: Type = find ...` |
| Outcome node (green/red) | `BranchResultNode`, `conclude: correct|error|...` |
| Aggregated branching (AND over branches) | `BranchAggregationNode`, `agg and|or ...` |
| Independent branching, "Undetermined" edge | aggregation with `mutex` / `hyp` operators; verify which one in the wiki before asserting |
| Iteration node (for all / while) | `CycleAggregationNode` (`cycle aggregation (exp) with Type item`) and `WhileCycleNode` (`while (exp)`) |
| Recursive node | procedure call / subinterpreter redirect: `ProcedureCallNode`, `BranchResultRedirectingNode`; see the wiki page on subinterpreters |
| Dictionaries (classes, properties, relationships, enums) | LOQI domain model: `class`, `enum`, `obj`, property and relationship declarations in `domain.loqi` |
| Problem formulation as an RDF knowledge graph | the *specific domain* LOQI file (or RDF via `rdf-to-domain-loqi`) passed to `reasoner-cli reason` |
| Solving tree / Verification tree / Hinting tree | one `tpg` type at runtime; the distinction is methodological, not enforced by the tools |
| Expression language (first-order logic over the graph) | LOQI expressions; the notation in the papers (`$X is operator`, `element::unused`, `$X->isInside($Y)`) is older than LOQI2 and must not be copied as current syntax |

## Facts Worth Quoting (with source)

- Development time: about 20 person-hours for the order-of-evaluation verification tree versus about 150 person-hours for the equivalent 230 handwritten Apache Jena rules (KII, p. 8; BICA, p. 38).
- Typical tree development: 10–50 person-hours per tree by developers who were not framework authors (KII, Table 1; BICA, p. 38).
- Domains modeled: order of expression evaluation, program tracing, variable scope and lifetime, access expressions, English noun-phrase word order (BICA, p. 38); KII lists four cores including name resolution.
- Performance: the interpreter is acceptable for human-sized problems but reaches about 2 s on realistic access-expression tasks; compiling to production rules is named as future work (KII, p. 8–9).
