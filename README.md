# DSH Workspace MCP

English | [简体中文](docs/README.zh.md)

DSH Workspace MCP loads MCP servers from your workspace configuration and makes their tools available to sessions opened in that workspace. Discovered workspaces stay mounted while the plugin runs, so switching workspaces keeps existing connections alive.

Requires DeepSeek Harness `0.2.2-rc.1` or later in the `0.2` series, plus Git, Node.js, and pnpm. [Download DeepSeek Harness](https://www.deepseek.com/harness/)

## Install

Install from GitHub. pnpm downloads the source and builds the loader and MCP Manager UI together.

```powershell
dsh plugin --profile web add github:yangjh0127/dsh-workspace-mcp#main --allow-build=dsh-workspace-mcp
```

Use the profile you run, such as `web` or `desktop`, then restart DSH. To update, rerun the install command and restart DSH.
For local development, see [linking a checkout](docs/guide/settings-ui.md#install).

## Configure MCP servers

- Use `.dsh/mcp.yml` or `.dsh/mcp.json` in the workspace root. The plugin also reads a workspace's `.mcp.json` for compatibility. Configuration changes reload automatically.
- Open **MCP Manager** from the main sidebar, Settings sidebar, or **Settings → Built-in plugins**. Use **Settings → General → MCP management page entries** to choose which shortcuts appear. Headless profiles use the loader and CLI.
- **Eager** is the default and mounts servers as soon as their workspace is discovered. Session creation and resume continue while Eager servers load. A model request can use the tools that are ready when it starts. Choose **Lazy** per server to connect on demand. In Lazy mode, use `mcp_search` to discover tools, `mcp_describe` to inspect their schemas, and `mcp_call` to invoke them. Connections stay open until the session ends or the server entry is unloaded.
- Workspace tools are scoped to the session's original workspace. User and profile configuration can provide shared servers across workspaces.

A `stdio` entry starts its configured command on the DSH host. Only enable servers and workspace configurations you trust.

## Documentation

- [Configuration format](docs/guide/format.md)
- [Configuration sources and precedence](docs/guide/layers.md)
- [Environment variable expansion](docs/guide/env-expansion.md)
- [CLI reference](docs/guide/cli.md)
- [Plugin service API](docs/guide/service.md)

Run `pnpm test` to build and test the plugin. Open bug reports in [Issues](https://github.com/yangjh0127/dsh-workspace-mcp/issues) and target `dev` for pull requests.

## License

[MIT](LICENSE)
