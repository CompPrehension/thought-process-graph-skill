# Source Discovery

Use this reference when a bundled-grammar fallback, local CLI runtime lookup, or implementation investigation requires finding sources or artifacts. An available MCP operation can be called directly from its description and schema; an unspecified version does not require source discovery before the call.

This reference describes how to find jars and sources. CLI examples in other references reuse the `DOMAIN_CLI_JAR` and `REASONER_CLI_JAR` variables defined here.

For simply running a supplied model, [model-execution.md](model-execution.md) is enough.

## Project Context First

1. Follow the user's paths and links and the project's instructions, README, build configuration, and dependency declarations for ITS projects, TPG model directories, `its_DomainModel`, and `its_Reasoner`. Inspect the likely files first; do not start with a repository-wide or drive-wide search.
2. If working inside `its_DomainModel`, `its_Reasoner`, or `its_QuestionGen`, use that checkout for implementation analysis. If a consuming project identifies a checkout or dependency version, use sources and artifacts matching that context rather than an unrelated newer installation. A repository URL identifies a project; it does not imply a local checkout exists.
3. If the required location is missing, check known local checkouts and installed sources/artifacts in the local Maven repository, narrowing by project/group/artifact name. Broaden incrementally only if these locations do not resolve the task. Discover only the required module; a DomainModel operation does not require a Reasoner jar.
4. For implementation analysis or unexplained behavior, locate matching sources. Use component versions already supplied by the project or tool metadata; do not infer a remote parser's version from the newest local jar. Routine remote execution does not require this investigation.

## Grammar Selection: Bundled First, Matching Sources If Needed

Most graph/domain work involves reading or writing LOQI. Start with [the bundled grammar](../assets/LoqiGrammar.g4), reading its header and the rules relevant to the task rather than the entire file. The header records the `its_DomainModel` version from which it was copied; it is a versioned snapshot, not a claim about every installed runtime.

1. Compare that header with the target `its_DomainModel` version if provided by the project/build configuration or runtime metadata. For MCP, the relevant parser version is the server's DomainModel dependency, not the locally installed Reasoner version. A tool schema describes its arguments and capabilities; it does not necessarily identify that dependency's version. Use information already available, without requiring a version search for ordinary execution.
2. When versions match and the required constructs are covered, use the snapshot for syntax lookup. When versions differ, the snapshot remains an orientation aid, but compatibility is unconfirmed. If the construct is missing, the snapshot does not explain the input, or the target parser disagrees, locate and analyze the target version's grammar and relevant builders. For a definitive claim about another version's syntax, confirm it against those sources.
3. Use the `its_DomainModel` checkout identified by the project context first. If none is identified or available, find installed source jars matching the target dependency through the Source Jars procedure below. Grammar locations are:
   - Checkout: `src/main/antlr4/its/model/definition/loqi/LoqiGrammar.g4` under `its_DomainModel`.
   - Source jar: `its/model/definition/loqi/LoqiGrammar.g4`.
4. If the target version is unknown, use the snapshot provisionally and validate with the actual MCP/CLI parser before claiming acceptance. If matching sources or runtime are unavailable, state the version/validation limit; do not claim that the snapshot proves compatibility.

Useful entry rules are `model` / `topDecl` for domain declarations, `fullTreeDecl` / `treeDecl` for a graph, and `fullExp` / `exp` for expressions. Read dependent rules as needed. Grammar describes syntax; builder validation, type rules, writer support, and runtime behavior may require matching source analysis even when parsing succeeds.

## Compatibility Baseline

The full functionality described by this skill is expected for `3.0` and newer artifacts of the CompPrehension projects. Still do not hard-code exact Maven versions in commands or instructions: discover the available artifacts first, then prefer the current `3.0+` artifact required by the consuming project or workspace.

## Maven Artifact Search

Artifacts, group `com.github.CompPrehension` (older installs may sit under `Max-Person`):

- `its_DomainModel`
- `its_Reasoner`
- `its_QuestionGen`

Use the version only after discovering it from an existing POM, source jar, or binary jar. Prefer the version identified by the project; otherwise choose the newest installed artifact compatible with the task.

## CLI Jars

Runnable CLI jars are the `*-all.jar` artifacts. For each required module, inspect the identified checkout's `target/` first, then the relevant artifact directory in the local Maven repository. For example, list DomainModel candidates in the current Maven group:

```bash
find "$HOME/.m2/repository/com/github/CompPrehension/its_DomainModel" \
  -name 'its_DomainModel-*-all.jar' 2>/dev/null
```

Use the configured local repository if it differs from `$HOME/.m2/repository`. For Reasoner, substitute `its_Reasoner`; check the older `Max-Person` group only if needed.

Rules for choosing:

1. Look for an `*-all.jar` under the project `target/` directories.
2. Look in the local Maven repository by artifact name, not by version.
3. If multiple versions exist, prefer the artifact required by the consuming project POM; otherwise prefer the newest discovered artifact that matches the task.
4. If no runnable artifact exists, run from the checkout with Maven, or ask the user to build/install the project locally:

```bash
(cd its_DomainModel && mvn -q exec:java -Dexec.args="--help")
(cd its_Reasoner && mvn -q exec:java -Dexec.args="--help")
```

After selection, set `DOMAIN_CLI_JAR` and/or `REASONER_CLI_JAR` to the chosen absolute jar paths, run `java -jar "$DOMAIN_CLI_JAR" --help` or `java -jar "$REASONER_CLI_JAR" --help` for the required module, and reuse the selection. Do not automatically choose the largest version filename over the project's required artifact. Use actual checkout paths for the Maven fallback above; ask for a build/install or jar only if no usable local option exists.

`its_QuestionGen` has no CLI in this skill's scope; use its library jar (`its_QuestionGen-*.jar`) and sources.

If the active local Maven repository belongs to a Windows user profile mounted in WSL, also try that path, for example `/mnt/c/Users/<USER>/.m2/repository`.

## Source Jars

Use checkout sources when identified by the project context. Otherwise look for a source jar matching the selected dependency or runtime artifact. Search the relevant artifact directory first, for example:

```bash
find "$HOME/.m2/repository/com/github/CompPrehension/its_DomainModel" \
  -name '*sources.jar' 2>/dev/null
```

Substitute the required artifact and configured Maven repository. Check the identified checkout's `target/` or older Maven group if needed; avoid searching all installed dependencies by default.

List and extract only what is needed:

```bash
jar tf path/to/its_DomainModel-*-sources.jar | rg 'CLI.kt|LoqiGrammar.g4|LoqiBuilder|LoqiWriter'
jar xf path/to/its_DomainModel-*-sources.jar src/main/kotlin/CLI.kt
jar xf path/to/its_DomainModel-*-sources.jar its/model/definition/loqi/LoqiGrammar.g4

jar tf path/to/its_Reasoner-*-sources.jar | rg 'misc/CLI.kt|DecisionTreeReasoner|ExpressionQuery'
jar xf path/to/its_Reasoner-*-sources.jar src/main/kotlin/misc/CLI.kt

jar tf path/to/its_QuestionGen-*-sources.jar | head -50
```

If no sources jar exists for an artifact, `unzip -l` the binary jar to learn package and class names, then read the matching workspace checkout.

## Workspace Checkouts

Use workspace checkouts identified by the project context first. If no location was supplied and installed artifacts did not resolve the task, check known workspace roots for these repository names:

```text
its_DomainModel/
its_Reasoner/
its_QuestionGen/
```

Key files:

```text
its_DomainModel/src/main/kotlin/CLI.kt
its_DomainModel/src/main/antlr4/its/model/definition/loqi/LoqiGrammar.g4
its_Reasoner/src/main/kotlin/misc/CLI.kt
```

## Upstream

The upstream organization is https://github.com/CompPrehension/. Use it as a reference link, but prefer local code for actual work unless the user explicitly asks for latest upstream behavior and internet access is available.

For published/user documentation, follow [docs-workflow.md](docs-workflow.md).
