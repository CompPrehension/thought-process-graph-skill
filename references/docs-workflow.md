# Documentation Workflow

Use this workflow when the agent or user has difficulty understanding how the concept is implemented in `its_DomainModel`, `its_Reasoner`, or `its_QuestionGen`: node behavior, model loading, traces, question generation, and related APIs. Select the page for the actual question using the map below.

Routine LOQI syntax lookup uses [the bundled grammar](../assets/LoqiGrammar.g4), graph/domain authoring uses [modeling-principles.md](modeling-principles.md), and deep scientific research uses [papers.md](papers.md). If the user ruled out web access, say which documentation you could not consult.

## Where The Documentation Lives

Published: `https://compprehension.github.io/docs/thought_process_graph/`

Sitemap: [https://compprehension.github.io/sitemap.xml](https://compprehension.github.io/sitemap.xml).

Source repository: `https://github.com/CompPrehension/CompPrehension.github.io`, branch `production`, directory `docs/decision_tree`. The pages are Markdown with Obsidian-style links and callouts. The text is in **Russian**; identifiers, file names, and code are Latin.

## Read Published Pages First

For ordinary documentation lookup, prefer reading the relevant published pages over cloning the repository:

1. Read the sitemap and select URLs under `/docs/thought_process_graph/`. Use the topic map below to identify the needed subject, then match it to the published URL. The module names, `LOQI`, `LOQI2`, `CLI`, and topic slugs help narrow the list.
2. Fetch and read the selected page, following relevant section links if needed. A sitemap lists URLs; it does not provide the documentation content needed to answer the question. Read only the pages relevant to the task.
3. Use the exact URLs from the sitemap or site links. Source paths under `docs/decision_tree` and published paths under `docs/thought_process_graph` differ; do not construct web URLs by translating Markdown file names.
4. If the sitemap cannot be read, navigate from the published documentation index or use a narrowly scoped site search. If published pages remain inaccessible but the source repository is reachable, use the local-copy fallback below.

Cite the published page when answering from it. For version-sensitive implementation questions, still confirm against the project's matching sources; a live documentation page does not establish that it describes the installed version.

## Map Of The Documentation

Use these topics to select a page in the sitemap. The paths below are a map of the Markdown sources, not published URLs; `$DOCS` denotes a local copy's `docs/decision_tree` directory when using the fallback. If the site layout changes, use the current sitemap and page links. For a local copy, inspect only the relevant topic directory if the mapped file is missing.

| Question | Page under `$DOCS/` |
|---|---|
| What the model is, how the three projects relate, installing via JitPack, Java vs Kotlin | `index.md` |
| Model directory layout, `BuildMethod`, tags, graph file naming, how a situation is built and merged | `1. Модели (its_DomainModel)/Модель решения задач (DomainSolvingModel).md` |
| Project overview of its_DomainModel | `1. Модели (its_DomainModel)/Об its_DomainModel.md` |
| Domain model concepts | `1. Модели (its_DomainModel)/Модель предметной области/` — `Модель предметной области.md`, `Классы.md`, `Свойства.md`, `Отношения.md`, `Объекты.md`, `Перечисления (Enum).md`, `Типы данных.md`, `Метаданные.md` |
| LOQI domain syntax | `1. Модели (its_DomainModel)/Модель предметной области/Представления данных/LOQI.md` |
| Legacy CSV dictionaries and RDF | `.../Представления данных/Словари + RDF.md` |
| Graph concept and semantics | `1. Модели (its_DomainModel)/Дерево решений/Дерево (граф) решений.md` |
| Node kinds and their behavior | `1. Модели (its_DomainModel)/Дерево решений/Узлы.md` |
| Aggregation methods (SEQ, MUTEX, SIM:AND/OR, HYP) and the feedback each supports | `1. Модели (its_DomainModel)/Дерево решений/Агрегация результатов и обратная связь.md` |
| Expressions inside graphs | `1. Модели (its_DomainModel)/Дерево решений/Выражения.md` |
| Metadata conventions (`id`, `skill`, `exception`, labels) | `1. Модели (its_DomainModel)/Дерево решений/Соглашения о метаданных.md` |
| Subinterpreters / calling one graph from another | `1. Модели (its_DomainModel)/Дерево решений/Субинтерпретаторы.md` |
| Graph representations (LOQI2, XML, draw.io) and conversions | `1. Модели (its_DomainModel)/Дерево решений/Представления дерева.md` |
| LOQI2 tree syntax (`tpg`, `ask`, `agg`, `merge`, fragments, lambdas) | `1. Модели (its_DomainModel)/Дерево решений/Способы описания/LOQI2.md` |
| Visual editor | `1. Модели (its_DomainModel)/Дерево решений/Способы описания/Визуальный редактор draw.io.md` |
| Reasoner overview | `2. Вычисления (its_Reasoner)/Об its_Reasoner.md` |
| Trace structure and reading | `2. Вычисления (its_Reasoner)/Трассы рассуждения.md` |
| Question generation (its_QuestionGen) | `3. Наводящие вопросы (its_QuestionGen)/` — `Об its_QuestionGen.md`, `Категории наводящих вопросов.md`, `Шаблонизация текстов.md` |
| CLI reference as published | `4. Инструменты/CLI.md` |
| JSON-RPC / MCP server | `4. Инструменты/JSON-RPC сервер.md` |
| This skill's own page | `4. Инструменты/Навык для LLM-агентов.md` |

## Optional Local Copy

Clone only when web reading is unsuitable or unavailable, or when repeated documentation work benefits from local full-text search and persistent/offline access. Reuse an existing clone identified in the project context after checking its repository, branch, and documentation directory; a separate skill-owned clone is not required.

For a new fallback clone, one possible project-local location is `.tmp/CompPrehension.github.io-production` (keep `.tmp/` ignored):

```bash
CLONE=.tmp/CompPrehension.github.io-production
if [ ! -d "$CLONE/.git" ]; then
  mkdir -p .tmp
  git clone --depth 1 --branch production \
    https://github.com/CompPrehension/CompPrehension.github.io.git "$CLONE"
fi
DOCS="$CLONE/docs/decision_tree"
```

For ongoing use, follow the project's chosen persistent checkout location instead. Refresh a clean checkout on the `production` branch with `git -C "$CLONE" pull --ff-only origin production` when fresh content is needed. Preserve local changes and other branches; do not reset the checkout to refresh it. If refresh fails, an existing copy remains usable, with its possible staleness noted.

## Searching A Local Copy

When the topic map does not settle the question and a local copy is in use, search the likely module/topic directory with Latin identifiers first (keywords, class names, CLI flags survive translation), then Russian terms. Broaden within `$DOCS` only if the scoped search fails. English words like "tree" or "graph" mostly hit code samples, not prose.

```bash
rg -n -i "merge|fragment|lambda" "$DOCS/1. Модели (its_DomainModel)/Дерево решений/Способы описания"
rg -n -i "наводящ|шаблон" "$DOCS/3. Наводящие вопросы (its_QuestionGen)"
```

Open only the specific pages needed for the task.

## If Documentation Cannot Be Read

If the relevant published pages and local documentation cannot be accessed:

1. State that the relevant documentation could not be read, mentioning any stale local copy used instead.
2. Fall back to project sources, the grammar selection procedure in [source-discovery.md](source-discovery.md), CLI help/source, and bundled references.
3. Do not claim documentation details were verified unless the actual page or local text was read, or supplied by the user.

Git is not needed for web reading. Network access is not needed to use an existing local copy.

## Handling The Temporary Clone

- Keep the clone under `.tmp/` or another clearly temporary project-local directory.
- Apply temporary-file cleanup to clones created only as transient fallbacks; retain a checkout intended for ongoing documentation access.
- Do not edit documentation files unless the user explicitly asks to modify documentation.
- Do not commit or treat the temporary clone as part of the target project.
