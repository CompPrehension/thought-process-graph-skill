# Thought Process Graph AI Agents Skill

This skill helps agents apply existing Thought Process Graphs, interpret their answers, and author, validate, or analyze TPG/LOQI models. It covers MCP/CLI usage, modeling methodology, and investigation of the `its_DomainModel`, `its_Reasoner`, and `its_QuestionGen` projects. Routing is adaptive by default and respects explicit capability, tool, source, and file-change limits. See [SKILL.md](SKILL.md) for the capability routes.

Runtime prerequisites depend on the selected task: use registered CompPrehension MCP tools, or build/install only the project required for an allowed CLI operation. Source discovery and version selection are described in [source-discovery.md](references/source-discovery.md). Text-only and conceptual work do not require installing the whole toolchain. When implementation documentation is needed and web access is allowed, use the [wiki](https://compprehension.github.io).

Checked tool versions: `its_DomainModel` (3.0.0-rc.7 and newer), `its_Reasoner` (3.0.0-rc.8 and newer), and `its_QuestionGen` (3.0.0-rc.7 and newer)
