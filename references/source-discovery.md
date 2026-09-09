# Source Discovery

Prefer the freshest local Maven artifact when available, because workspace checkouts can lag behind the installed dependency used by another project. Do not pin versions in instructions: versions can change. Search by package/group and artifact names, then choose the relevant/current artifact from what exists locally.

This is the single place in the skill that describes how to find jars and sources. Other references reuse the `DOMAIN_CLI_JAR` and `REASONER_CLI_JAR` variables defined here.

## Compatibility Baseline

The full functionality described by this skill is expected for `3.0` and newer artifacts of the CompPrehension projects. Still do not hard-code exact Maven versions in commands or instructions: discover the available artifacts first, then prefer the current `3.0+` artifact required by the consuming project or workspace.

## Maven Artifact Search

Artifacts, group `com.github.CompPrehension` (older installs may sit under `Max-Person`):

- `its_DomainModel`
- `its_Reasoner`
- `its_QuestionGen`

Use the version only after discovering it from an existing POM, source jar, or binary jar.

## CLI Jars

Runnable CLI jars are the `*-all.jar` artifacts. Discover them once per session and reuse the variables in every command:

```bash
DOMAIN_CLI_JAR="$(find its_DomainModel/target "$HOME/.m2/repository" -name 'its_DomainModel-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"
REASONER_CLI_JAR="$(find its_Reasoner/target "$HOME/.m2/repository" -name 'its_Reasoner-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"

java -jar "$DOMAIN_CLI_JAR" --help
java -jar "$REASONER_CLI_JAR" --help
```

Rules for choosing:

1. Look for an `*-all.jar` under the project `target/` directories.
2. Look in the local Maven repository by artifact name, not by version.
3. If multiple versions exist, prefer the artifact required by the consuming project POM; otherwise prefer the newest discovered artifact that matches the task.
4. If no runnable artifact exists, run from the checkout with Maven, or ask the user to build/install the project locally:

```bash
(cd its_DomainModel && mvn -q exec:java -Dexec.args="--help")
(cd its_Reasoner && mvn -q exec:java -Dexec.args="--help")
```

`its_QuestionGen` has no CLI in this skill's scope; use its library jar (`its_QuestionGen-*.jar`) and sources.

If the active local Maven repository belongs to a Windows user profile mounted in WSL, also try that path, for example `/mnt/c/Users/<USER>/.m2/repository`.

## Source Jars

Use targeted searches. Avoid broad full-drive scans unless necessary.

```bash
find "$HOME/.m2/repository" -path '*its_DomainModel*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*its_Reasoner*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*its_QuestionGen*' -name '*sources.jar'
find its_DomainModel/target its_Reasoner/target its_QuestionGen/target -name '*sources.jar' 2>/dev/null
```

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

If source jars are absent, inspect workspace repositories:

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
