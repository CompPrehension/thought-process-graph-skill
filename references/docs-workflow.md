# Documentation Workflow

Use this workflow whenever a task needs CompPrehension user documentation for thought process graphs, LOQI, the model directory layout, traces, or question generation.

## Where The Documentation Lives

Published: `https://compprehension.github.io/docs/thought_process_graph/`

Source repository: `https://github.com/CompPrehension/CompPrehension.github.io`, branch `production`, directory `docs/decision_tree`. The pages are Markdown with Obsidian-style links and callouts. The text is in **Russian**; identifiers, file names, and code are Latin.

## Get A Local Copy

The skill does not depend on any checkout lying next to the project; it needs a clone of its own. Use one fixed location inside the current project workspace, `.tmp/CompPrehension.github.io-production`, so that repeated tasks in the same project reuse it. Check for it first, refresh it if present, clone it if not. Do not rely on a browser-only lookup or on memory.

```bash
CLONE=.tmp/CompPrehension.github.io-production
if [ -d "$CLONE/.git" ]; then
  git -C "$CLONE" fetch --depth 1 origin production && git -C "$CLONE" reset -q --hard origin/production
else
  mkdir -p .tmp
  git clone --depth 1 --branch production \
    https://github.com/CompPrehension/CompPrehension.github.io.git "$CLONE"
fi
DOCS="$CLONE/docs/decision_tree"
```

If the refresh fails (offline), keep using the existing clone and say that it may be stale.

## Map Of The Documentation

The tree is small (about 30 pages) and its layout is stable. Go straight to the page for the question instead of grepping for keywords. Confirm the current list with `find "$DOCS" -name '*.md'` once per session; page titles below are the actual file names.

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

## Searching Inside The Docs

When the map does not settle the question, search with Latin identifiers first (keywords, class names, CLI flags survive translation), then Russian terms. Note that English words like "tree" or "graph" mostly hit code samples, not prose.

```bash
rg -n -i "merge|fragment|lambda|subinterpreter|BranchResultNode" "$DOCS"
rg -n -i "мыслительн|рассужд|наводящ|метаданн|субинтерпретатор" "$DOCS"
```

Open only the specific pages needed for the task.

## If Network Or Git Is Unavailable

If cloning fails because network access is unavailable or the repository cannot be reached:

1. Say that the current documentation could not be cloned.
2. Fall back to local project sources, `LoqiGrammar.g4`, CLI help/source, and bundled references.
3. Avoid claiming documentation details as current unless they were read from a local clone or provided by the user.

## Handling The Temporary Clone

- Keep the clone under `.tmp/` or another clearly temporary project-local directory.
- Do not edit documentation files unless the user explicitly asks to modify documentation.
- Do not commit or treat the temporary clone as part of the target project.
