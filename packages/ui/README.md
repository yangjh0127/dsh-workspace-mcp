# dsh-workspace-mcp-ui

Three entries for [dsh-workspace-mcp](https://github.com/yangjh0127/dsh-workspace-mcp) — a main sidebar row and page, a settings sidebar section, and the Built-in plugins settings tab — all opening the same panel. It lists the servers the loader will run, and writes only the managed `mcp.yml` for that scope. A same-name copy in another file stays as a note on that card. A non-yml row with a different name but the same command and arguments, or the same URL, is left off the list. Add MCP opens a form and writes the row. Fill from clipboard reads one JSON or YAML server into that form. MCP config stays beside it and opens the file for anything the form does not cover. The page opens on a **Current workspace** header (name, path, server counts) and pins that workspace group. **Settings → General → MCP management page entries** chooses which of the three entries appear; the same row is also shown on the manager bundle's page in the plugin manager. The main page's back button returns to the conversation without changing the session.

The root package ships this UI with the loader. Follow the [root installation guide](../../README.md#install).

The page has three entries, all enabled by default: the **MCP Manager** row in the main sidebar, the **MCP Manager** section in the settings sidebar, and the **Plugin: MCP Manager** settings tab. All render the same panel. **Settings → General → MCP management page entries** switches each one on or off; turning all three off is allowed, and that row always stays. In the plugin manager, switch off the manager bundle's `mcp-project-ui` component to hide all three entries at once, leaving `mcp-project` enabled; a profile patch override disabling that UI row also works. The loader keeps running. In headless mode there is no browser runtime or page. Without the dsh `connection` service, no UI routes are registered; its connection-scoped integration waits without blocking the core loader. The UI row itself can be active, and the connection integration is registered and cleaned up with that service's lifetime.

For local development, run `pnpm install` and `pnpm run build` in the repository root, then `dsh plugin --profile web add link:<source>`. The root build compiles core first and then the UI (host code and browser bundle); only the manager link is needed. Rebuild after source changes.

If the UI was installed separately, first install the manager bundle, then remove `dsh-project-mcp-ui` from the profile bundle selection (`dsh.profile.bundles`), keeping the manager selected. Optionally run `dsh plugin --profile web remove dsh-project-mcp-ui` afterwards to remove the direct UI dependency (use `desktop` for that profile); the manager's transitive UI dependency remains. Old bundle selections and direct dependencies are not promised to be cleaned up automatically.

If the old standalone UI bundle was disabled, preserve that intent **before loading the new manager bundle** with this profile patch row override; disabling the old bundle alone does not stop the new default-enabled UI component:

```yaml
- id: mcp-project-ui
  disabled: true
```

Usage guide: [MCP Manager page](https://github.com/yangjh0127/dsh-workspace-mcp/blob/main/docs/guide/settings-ui.md) ([中文](https://github.com/yangjh0127/dsh-workspace-mcp/blob/main/docs/guide/settings-ui.zh.md)).
