# Source Discovery

Prefer the freshest local Maven artifact when available, because workspace checkouts can lag behind the installed dependency used by another project. Do not pin versions in instructions: versions can change. Search by package/group and artifact names, then choose the relevant/current artifact from what exists locally.

## Compatibility Baseline

The full functionality described by this skill is expected for `3.0` and newer artifacts of the CompPrehension projects. Still do not hard-code exact Maven versions in commands or instructions: discover the available artifacts first, then prefer the current `3.0+` artifact required by the consuming project or workspace.

## Maven Artifact Search

Search for these artifact names without assuming a version:

- `its_DomainModel`
- `its_Reasoner`
- `its_QuestionGen`

Likely group path fragments:

- `CompPrehension`
- `Max-Person`

Use the version only after discovering it from an existing POM, source jar, or binary jar.

## Find Source Jars

Use targeted searches. Avoid broad full-drive scans unless necessary.

Linux/WSL/macOS:

```bash
find "$HOME/.m2/repository" -path '*its_DomainModel*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*its_Reasoner*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*its_QuestionGen*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*CompPrehension*' -name '*sources.jar'
find "$HOME/.m2/repository" -path '*Max-Person*' -name '*sources.jar'
```

Windows PowerShell:

```powershell
$m2 = "$env:USERPROFILE\.m2\repository"
Get-ChildItem $m2 -Recurse -File -Filter "*sources.jar" -ErrorAction SilentlyContinue | Where-Object { $_.FullName -match "its_DomainModel|its_Reasoner|its_QuestionGen|CompPrehension|Max-Person" }
```

If the active local Maven repository is on Windows/WSL, also try the specific user repository path if known, for example:

```bash
find /mnt/c/Users/<USER>/.m2/repository -path '*its_DomainModel*' -name '*sources.jar'
```

## Inspect Source Jars

List and extract only what is needed:

```bash
jar tf path/to/its_DomainModel-*-sources.jar | rg 'CLI.kt|LoqiGrammar.g4|LoqiBuilder|LoqiWriter'
jar xf path/to/its_DomainModel-*-sources.jar src/main/kotlin/CLI.kt
jar xf path/to/its_DomainModel-*-sources.jar its/model/definition/loqi/LoqiGrammar.g4

jar tf path/to/its_Reasoner-*-sources.jar | rg 'misc/CLI.kt|DecisionTreeReasoner|ExpressionQuery'
jar xf path/to/its_Reasoner-*-sources.jar src/main/kotlin/misc/CLI.kt
```

The workspace may also contain built source jars under `target/`:

```bash
find its_DomainModel/target its_Reasoner/target its_QuestionGen/target -name '*sources.jar' 2>/dev/null
```

PowerShell:

```powershell
Get-ChildItem its_DomainModel\target,its_Reasoner\target,its_QuestionGen\target -Recurse -File -Filter "*sources.jar" -ErrorAction SilentlyContinue
```

## Fallback To Workspace Checkouts

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

## CLI Artifacts

If CLI tasks require a jar:

1. Look for an `*-all.jar` under the project `target/` directories.
2. Look in the local Maven repository by artifact name, not by version.
3. If multiple versions exist, prefer the artifact required by the consuming project POM; otherwise prefer the newest discovered artifact that matches the task.
4. If no runnable artifact exists, ask the user to build/install the project locally.

Useful searches:

Linux/WSL/macOS:

```bash
find its_DomainModel/target "$HOME/.m2/repository" -name 'its_DomainModel-*-all.jar' 2>/dev/null
find its_Reasoner/target "$HOME/.m2/repository" -name 'its_Reasoner-*-all.jar' 2>/dev/null
find "$HOME/.m2/repository" -name 'its_QuestionGen-*.jar' 2>/dev/null
```

Windows PowerShell:

```powershell
$m2 = "$env:USERPROFILE\.m2\repository"
Get-ChildItem its_DomainModel\target,$m2 -Recurse -File -Filter "its_DomainModel-*-all.jar" -ErrorAction SilentlyContinue
Get-ChildItem its_Reasoner\target,$m2 -Recurse -File -Filter "its_Reasoner-*-all.jar" -ErrorAction SilentlyContinue
Get-ChildItem $m2 -Recurse -File -Filter "its_QuestionGen-*.jar" -ErrorAction SilentlyContinue
```

For CLI examples, assign discovered jars to variables:

Linux/WSL/macOS:

```bash
DOMAIN_CLI_JAR="$(find its_DomainModel/target "$HOME/.m2/repository" -name 'its_DomainModel-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"
REASONER_CLI_JAR="$(find its_Reasoner/target "$HOME/.m2/repository" -name 'its_Reasoner-*-all.jar' 2>/dev/null | sort -V | tail -n 1)"
```

Windows PowerShell:

```powershell
$DOMAIN_CLI_JAR = Get-ChildItem its_DomainModel\target,$env:USERPROFILE\.m2\repository -Recurse -File -Filter "its_DomainModel-*-all.jar" -ErrorAction SilentlyContinue | Sort-Object LastWriteTime | Select-Object -Last 1 -ExpandProperty FullName
$REASONER_CLI_JAR = Get-ChildItem its_Reasoner\target,$env:USERPROFILE\.m2\repository -Recurse -File -Filter "its_Reasoner-*-all.jar" -ErrorAction SilentlyContinue | Sort-Object LastWriteTime | Select-Object -Last 1 -ExpandProperty FullName
```

## Upstream

The upstream organization is https://github.com/CompPrehension/. Use it as a reference link, but prefer local code for actual work unless the user explicitly asks for latest upstream behavior and internet access is available.

For published/user documentation, do not rely on a browser-only lookup. Follow [docs-workflow.md](docs-workflow.md): clone `CompPrehension.github.io` branch `production` into a temporary project-local folder and search `docs/decision_tree`.
