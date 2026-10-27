# Modeling Graphs And Domain Data

Read this when creating a TPG from scratch, adding new reasoning or domain data to a graph, reviewing an existing graph, or explaining a methodological choice. The core Rationale is in [SKILL.md](../SKILL.md). Routine syntax lookup starts with [the bundled grammar](../assets/LoqiGrammar.g4) and its version header. Deep scientific or pedagogical research requires reading the original articles via [papers.md](papers.md); routine modeling decisions use this guide.

**New parts versus existing ones.** When you build a graph or add something new, apply the guide directly. In parts of a graph that already exist, the current structure reflects the author's pedagogical decisions: the order of checks, how errors are split into outcomes, which feedback each path produces. A change that looks methodologically better can still change what students are told, or break question generation and metadata that depend on that structure. So report methodological findings about existing parts as proposals: what you noticed, the principle behind it, and the concrete change. Apply them only after the user agrees, even when you are already editing that graph for another reason. Defects found by the tools (syntax, types, loading errors) and the changes the user explicitly asked for are not covered by this and can be fixed as usual.

## Sources And Scope

This guide condenses the authors' methodology from two bundled papers:

- **BICA**: [Educational Models for Cognition](../assets/bica-2023-educational-models-for-cognition.pdf), the April 2025 preprint. The main sources are sections 3, 4, 5.1, and 7.
- **KII**: [Фреймворк для разработки интеллектуальных обучающих систем на основе моделей предметных областей в виде деревьев решений](../assets/kii-2023-thought-process-graph.pdf), 2023, especially section 1.

Page references below are **one-based PDF pages**, including the BICA highlights sheet; they differ from the printed page numbers. The papers call the model a decision tree or Thought Process Tree (TPT). This guide uses the current name TPG for the same methodological discussion.

This guide explains *why* to choose a structure and names the current construct that expresses it. That is enough to design a graph. When the exact behavior of a construct matters (result combination rules, required outgoing branches, validation errors), check it the cheapest way available: run the model through the tools ([model-execution.md](model-execution.md), [cli-guide.md](cli-guide.md)), look at [the grammar](../assets/LoqiGrammar.g4), or read the implementation ([source-discovery.md](source-discovery.md), [docs-workflow.md](docs-workflow.md)).

## Decide What The Graph Teaches And Does

Identify the problem-solving step and the reasoning substeps being modeled. One tree usually represents performing or checking one step over a formal task situation, with objects, properties, and relationships. Its organization determines which hints, explanations, and pedagogical questions the tutor can provide. (BICA section 3, PDF pp. 9–16; KII section 1, pp. 4–5.)

- **Solving/hinting**: find a correct action, generate a worked example or next-step hint, and explain why it is correct. Describing one correct solution path can suffice for this purpose.
- **Verification**: check the student's proposed step, classify violated constraints, and explain why that proposal is wrong in this situation. Account for the valid alternatives in the modeled task; matching one preferred solution path is insufficient when other paths are correct.

Both kinds can support pedagogical dialogue. Their distinction describes the purpose of the model; the papers do not prescribe separate current runtime types. Solving a problem by searching with a verification tree can be inefficient, comparable to brute-force search and backtracking. (BICA section 3, PDF pp. 13–16.)

## Design The Domain Data Alongside The Graph

### Concepts, properties, and relationships

Use the dictionaries described in BICA section 3 (PDF pp. 11–14) to determine the information the reasoning needs:

| Domain element | Modeling role |
|---|---|
| Classes and inheritance | Kinds of objects in the problem formulation and solution. The paper recommends introducing a class when a property or relationship needs to be restricted to objects of that class. |
| Class properties | Values shared by objects of a class; a property declared in a parent class can take different values in subclasses. |
| Object properties | Values that can differ between individual objects of the same class. |
| Property types | Boolean, numeric, enumerated, and comparison-scale values, chosen according to the knowledge the reasoning uses. |
| Relationships | Named connections between objects with restrictions on participating classes; ordering relationships can support derived comparisons such as before, between, or closest. |
| Question and action vocabulary | Reusable kinds of questions and actions, avoiding several slightly different formulations of the same operation. |

The paper's dictionaries become a domain ontology in formal execution, allowing type checks on expressions, properties, and relationship participants. In current LOQI they are `class`, `enum`, `class prop` / `obj prop`, and `rel` declarations in `domain.loqi`; the task situation is a separate LOQI file with objects. (BICA section 5.1, PDF p. 36.)

### Represent the distinctions the learner must reason about

Keep formally different entities distinct. In the expression-evaluation example, tokens form the visible notation, while operators are evaluated entities. One operator can consist of multiple tokens, so the model represents both kinds and connects them with a relationship. (BICA section 4.1, PDF pp. 19–22.)

Choose data representations that reflect the expected knowledge. The same example uses a comparison scale for precedence: the learner needs to compare precedence, without memorizing numerical precedence levels. In current LOQI an ordering is usually a relationship with a `linear` or `partial` scale, with dependent relationships (`opposite`, `transitive`, `between`, `closer`, `further`) derived from it. (BICA section 4.1, PDF pp. 19–21.)

Express relevant behavior through properties when that is what explains the rule. In the paper's example, a class property describing when an operand is evaluated makes the reasoning reusable across operators and programming languages. Questions based only on operator names instead teach a name-to-behavior association and require tree changes when that association differs. This recommendation concerns behavior-based reasoning; it does not forbid class distinctions needed by the domain. (BICA section 4.2.3, PDF pp. 28–31.)

### Respect the information available to the learner

Constraints should operate on the formal problem formulation and the student's answer so that explanations refer directly to the task's objects and rules. Using an independently obtained correct answer as the main basis of verification can produce feedback that merely states what the answer should have been. (BICA section 4, PDF p. 17; section 4.3.1, p. 32.)

Intermediate variables model the learner's short-term memory. The author explicitly rules out constructing and remembering intermediate arrays of found objects for later processing in the modeled reasoning. Use iteration to find and process objects individually. Collections in the problem formulation or answer are allowed because they remain visible to the learner. This is a pedagogical restriction, not a claim that the implementation cannot store collections. (BICA section 4.2.1, PDF p. 25.)

## Choose Nodes From The Reasoning Need

The following choices summarize BICA section 3 (PDF pp. 10–15). The dependency rule for sequential questions is developed in sections 4.2.2 and 4.3.2 (pp. 27–29, 33–35).

| Reasoning need | Paper construct | Current LOQI2 | Selection criterion |
|---|---|---|---|
| Ask a question whose answer determines the next reasoning step | Question node | `ask (...)`, `ask tuple (...)` | Outgoing cases correspond to Boolean, limited numeric, or enumerated answers. |
| Find a relevant object or calculate a value needed later | Action node | `var V: T = find ...` | Assign the result to a variable; represent the not-found case when a search may fail. |
| Conclude a subproblem or identify an error | Outcome node | `conclude: correct / error / null` | Verification outcomes distinguish success from a specific kind of error; the path supplies the context for its explanation. |
| Combine conclusions of several independent subproblems | Aggregated branching | `agg and` / `agg or` | Use a Boolean combination matching their joint meaning; the paper usually uses AND for required constraints. |
| Examine alternative cases where at most one branch can give an outcome | Independent branching | `agg mutex` | Case order is immaterial; an Undetermined (`null`) continuation handles the absence of a conclusion from all cases. |
| Repeat reasoning for objects satisfying criteria, or until a condition holds | Iteration node | `cycle <agg> (...) with T V`, `while (...)` | Express the repeated reasoning and its continuation; avoid imposing an object order when the reasoning does not require one. |
| Apply the same reasoning to another object/subproblem | Recursive node | `subcall(...)` / `conclude: subcall(...)` to another graph | Use it for genuinely recursive reasoning, with conditions that avoid unending recursion. |

### Aggregation, alternative cases, and sequential dependence

These structures express different relationships between checks. Choose by that relationship before selecting a current language construct:

- **Several constraints must all hold**: aggregate their conclusions with AND. The worked example combines four branches checking whether different operands or evaluation-order restrictions prevent an operator's evaluation. More than one constraint can matter; the branches are not mutually exclusive cases. (BICA section 4.1, PDF pp. 19–20.)
- **At most one case provides the conclusion**: use independent branching. The paper's example checks three possible parentheses/enclosing-operator cases between competing operators. Their checking order does not matter. (BICA sections 4.1 and 4.2.2, PDF pp. 20, 27–29.)
- **An earlier answer determines whether a later question is needed**: use sequential questions. The example checks parentheses before precedence and considers associativity only when precedence is equal. This order expresses domain reasoning. (BICA section 4.2.2, PDF p. 27.)

Unnecessarily chaining independent questions can make a pedagogical dialogue insist on one reasoning order and reject another valid one. The author gives the general criterion that two questions should be sequential only when the need to ask the second depends on the answer to the first. (BICA sections 4.2.2 and 4.3.2, PDF pp. 27–29, 35.)

The current aggregation methods carry these ideas, each with a pedagogical purpose. A branch result is `true`/`correct`, `false`/`error`, or `null` (correctness not established):

- **`agg and` (SIM:AND)**: several independent constraints must all hold. Each violated constraint yields its own error explanation, so the student sees every problem, not only the first one found.
- **`agg or` (SIM:OR)**: any one of several independent conditions is sufficient. On error, the explanation has to address all conditions together, or ask which one the student relied on.
- **`agg mutex`**: alternative cases where at most one can conclude, the paper's independent branching. Order does not matter; when no case applies, the result is `null`, and a `null` continuation lets reasoning go on.
- **Sequence** (questions chained through answers, or `while`): a later check is needed only for some answers of an earlier one. Use it only for genuine dependence.
- **`agg hyp`**: hypotheses about how the student could have reached their answer when the answer alone does not reveal it. It supports a clarifying question instead of guessing the error, which addresses the diagnostic-ambiguity limit below.

## Build Verification Around The Student's Proposal

The methodological goal is a set of constraints under which every correct answer passes and every incorrect answer violates a constraint. This requires modeling the domain's correctness rules over the problem formulation. It is more demanding than computing one correct answer and comparing the student's answer to it. (BICA section 4, PDF p. 17.)

For tasks with one correct answer, keep the explanation focused on the proposed action and its violated rule. In the paper's sequence-execution example, the checks establish whether the selected statement was already executed and whether an earlier statement remains unexecuted. They yield specific reasons tied to the student's choice. (BICA section 4.3.1, PDF p. 32.)

The author recommends beginning with the hypothesis that the student's proposal could be correct, then checking the conditions that permit it, from broader conditions to narrower ones. This gives more informative errors than first deriving the correct action and checking the proposal only at the end. Preserve genuine dependencies between questions while keeping independent checks independent. (BICA section 4.3.2, PDF pp. 32–35.)

A correctly built verification tree classifies errors through its error outcomes; the path to an outcome assigns the variables needed for a concrete explanation. Assess the graph by the explanations and pedagogical questions its structure supports as well as by the correctness result. (BICA section 3, PDF pp. 15–16; KII section 1, pp. 4–5.)

## Applicability And Limits

- The demonstrated scope is relatively simple intellectual skills requiring reasoning with well-defined rules. The reported domains include programming topics and English noun-phrase word order. Extending the method to invention, creativity, and broader cognitive tasks remains a research question in the paper. (BICA sections 6–7, PDF pp. 38–41.)
- The student's answer may lack enough information to identify the underlying error or even establish an error in their thinking. The paper treats forming and testing hypotheses through dialogue as further work. A practical consequence is to retain diagnostic ambiguity when the available information does not distinguish explanations. (BICA section 7, PDF p. 40.)
- Formal execution requires formalized task data, dictionaries, and node/edge expressions. Natural-language trees can support human modeling and instruction, but executable semantics require this additional formalization. (BICA sections 5–5.2, PDF pp. 35–37.)
- Performance measurements and proposed compiler work describe the historical implementations studied in the papers, not today's runtime. The papers themselves leave the sufficiency of trees versus graphs open. (BICA sections 5.3 and 7, PDF pp. 37, 40; KII sections 3–4, pp. 8–10.)

## Implement And Check With The Current Toolchain

If you have not built a TPG model before and it is unclear how the domain, graph, and situation files fit together, start from [minimal-example.md](minimal-example.md): a small validated model with expected results that you can copy and grow. Skip it once the structure is clear.

Write the LOQI using [the bundled grammar](../assets/LoqiGrammar.g4) and [loqi-tpg-reference.md](loqi-tpg-reference.md). Validate the domain and graph, then run one representative situation per expected outcome through the tool selected in [SKILL.md](../SKILL.md). Then review whether outcomes explain the student's proposal, valid alternatives remain accepted, and ordering reflects genuine dependencies. Parser/type validation and methodological review establish different properties. If the user asked only for a draft or a review, say which of these checks you did not run.

For full worked trees, diagrams, bibliographic details, or empirical comparisons, use [papers.md](papers.md) to select the relevant source pages. There is no need to reopen the papers for the recommendations already summarized here.
