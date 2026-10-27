# Connect The CompPrehension MCP Server

This is a conditional reference for MCP integration problems. For an available tool, use its description and schema directly.

## When To Read This Reference

Read only the relevant section in these cases:

| Trigger | What this reference helps resolve |
|---|---|
| The user asks to install, start, register, or connect the server | Server identification, launch, and client registration. |
| Diagnosing missing tools, connection failure, or authentication failure is in scope | Whether the server is running, the client registration is correct, and the configured connection/authentication is usable. |
| File transfer or path access fails and the tool schema does not settle the issue | Whether the server can access the supplied files or needs a supported inline payload. |

Routine model execution, validation, and conversion do not require this reference. Missing version metadata alone is not a trigger. Neither is a missing MCP tool during ordinary work: fall back to the CLI and investigate the connection only when the user asks for it. An unexpected graph result or a model syntax/type error belongs to model analysis or debugging, not MCP setup.

## Identify And Connect The Server

- Repository: [CompPrehension-Toolchain-Server](https://github.com/CompPrehension/CompPrehension-Toolchain-Server).
- README: [build, run, configuration, and MCP connection](https://github.com/CompPrehension/CompPrehension-Toolchain-Server/blob/main/README.md).

The server is commonly checked out as `compph-toolchain-server`. It exposes DomainModel, Reasoner, and MeaningTree operations. For the integration issue being investigated, read the relevant section of the identified checkout/version's README, or the upstream README above when no matching local documentation is available. It covers build/run requirements, authentication, file payloads, and the [MCP endpoint](https://github.com/CompPrehension/CompPrehension-Toolchain-Server#mcp-endpoint-post-mcp). These links are for the triggered setup/diagnostic task, not preliminary reading before ordinary calls.

1. Use the server checkout or installation identified by the project context. Build or install it only if the user wants the server set up.
2. Start the built server according to its README. The existing executable-jar form is:

   ```bash
   java -jar target/compph-toolchain-server-<version>.jar [PORT]
   ```

3. Register its `http://localhost:<port>/mcp` endpoint as an HTTP MCP server in the agent client when running locally. Use the configured address for a remote installation. The server uses Streamable HTTP; consult the client's own setup instructions for the configuration syntax.
4. Confirm that the client exposes the required operations, such as `reasoner__reason` or `domain__validate-dsm`. Client-specific prefixes or name normalization may apply. A reachable endpoint alone does not establish that the tools are registered and callable by the agent.

## Connection And Data Access Checks

If the expected tools are missing, check the client registration and the server's startup output. Do not replace a missing MCP tool with hand-written HTTP/JSON-RPC requests during ordinary model work; the CLI is the usual fallback.

File arguments follow each exposed tool's schema. Paths on the client machine are not generally server-side paths; the server's local-path mode requires `LOCAL_FILES_DISCOVERY=true` and access to those files. Prefer supported inline payloads when shared filesystem access is unavailable.

Use any component/version metadata the server actually provides. The server package version or its input schema alone need not identify the embedded DomainModel grammar version. Investigate that dependency only when a requested version check or implementation diagnosis requires it.
