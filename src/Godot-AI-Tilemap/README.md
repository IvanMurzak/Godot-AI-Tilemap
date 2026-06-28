# Tilemap Tools

AI MCP tools for Godot Tilemap.

A **source-only** MCP tool extension for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP).
The package ships C# source (no compiled DLL, no bundled Godot) that compiles inside your Godot project
against your own GodotSharp, so it never locks you to a Godot version.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.Tilemap

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.Tilemap" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool families are auto-discovered — no registry edit.

## Tools

All tools wrap Godot's built-in `TileMapLayer` node (Godot 4.3+, the modern replacement for the
deprecated `TileMap`). Every tool is editor-only and addresses a layer by its node path relative to the
edited scene root.

| Tool | Description |
| --- | --- |
| `tilemap-create` | Create a `TileMapLayer` node (optional name / parent). |
| `tilemap-set-tileset` | Assign a `TileSet` resource to a layer (by `res://` path). |
| `tilemap-set-cell` | Set a single cell (coords + TileSet source id + atlas coords). |
| `tilemap-erase-cell` | Erase a single cell. |
| `tilemap-get-used-cells` | List a layer's used cells (read-only). |
| `tilemap-clear` | Clear all cells on a layer. |

License: Apache-2.0.
