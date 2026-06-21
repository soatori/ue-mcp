# Unreal MCP Configure Workflow

Use this reference when handling `/unreal-mcp:configure <target>` or `/ue-mcp:configure <target>`.

## Command

The agent should invoke this workflow for `/unreal-mcp:configure <target>`. The bundled helper is implementation support; ordinary users should call the skill command instead of manually starting the helper.

```bash
python scripts/configure-unreal-mcp.py -ProjectPath "/path/to/Project" -Target all -Port 8000 -DryRun
```

Targets are `claude`, `codex`, `cursor`, `vscode`, `gemini`, and `all`.

Important switches:

- `-DryRun` or `--dry-run`: print planned file changes without writing. Use this first.
- `-EnablePlugins` or `--enable-plugins`: add the core `ModelContextProtocol` and `ToolsetRegistry` entries to the `.uproject`.
- `-AutoStart` or `--auto-start`: write project default MCP settings to `Config/DefaultEngine.ini`.
- `-Verify` or `--verify`: try a short HTTP request to `http://127.0.0.1:<Port>/mcp`.
- `-Target all` or `--target all`: enables the core MCP plugins, writes Auto Start defaults, and configures all supported clients.

The script is Python and cross-platform. It accepts the UE-style `-ProjectPath`, `-Target`, `-Port`, `-DryRun`, `-EnablePlugins`, `-AutoStart`, and `-Verify` flags, plus lowercase GNU-style aliases.

## What The Script Changes

- Resolves exactly one `.uproject` from `-ProjectPath`.
- Enables the core Unreal MCP plugins: `ModelContextProtocol` and `ToolsetRegistry`.
- Writes project default settings for Auto Start, port, URL path, and Tool Search.
- Merges JSON client configs for Claude Code, Cursor, VS Code, and Gemini without deleting existing MCP servers.
- Creates Codex `.codex/config.toml` only when it does not already exist.

After the core connection is configured, enable Toolset plugins according to the task. `AllToolsets` is acceptable for broad exploration or prototyping when the user accepts the larger startup and schema surface; otherwise enable the specific domain Toolsets needed and verify them with `list_toolsets`.

## Codex TOML Rule

Codex TOML generation is write-once. If `.codex/config.toml` exists, stop and ask the user whether to edit or delete it. Do not overwrite it automatically.

## Editor Fallback

If project defaults are ignored by a specific engine build, use the editor UI:

1. Enable **Unreal MCP** in Edit > Plugins.
2. Open Editor Preferences > General > Model Context Protocol.
3. Enable Auto Start Server.
4. Set port `8000` and URL path `/mcp`.
5. Run `ModelContextProtocol.GenerateClientConfig <Client|All>` from the editor console.
6. Start the agent from the project root and call `list_toolsets`.

## Verification

After configuration:

1. Launch or restart the UE editor.
2. Confirm Output Log includes `LogModelContextProtocol` startup messages.
3. Start the agent from the project root where config was written.
4. Call `list_toolsets`.
5. If Toolsets are missing, run `ModelContextProtocol.RefreshTools`, reconnect the agent, and inspect `LogModelContextProtocol`.

