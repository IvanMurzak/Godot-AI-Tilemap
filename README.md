<h1 align="center">Godot AI Tilemap</h1>

<p align="center">
  AI <b>MCP tools</b> for Godot's built-in <b>TileMapLayer</b> — an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-Tilemap` is a focused Godot-MCP extension for Godot's built-in `TileMapLayer` node (Godot
4.3+, the modern replacement for the deprecated `TileMap`). It adds an MCP tool family for creating
tile-map layers, assigning a `TileSet`, and painting / erasing / reading individual cells — authored in
C# with `[AiToolType]` / `[AiTool]` (the same model as Unity-MCP and the core Godot-MCP addon), and
shipped as a **source-only NuGet package** that compiles inside any consumer's Godot project against the
consumer's own GodotSharp — no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `tilemap-create` | editor (`#if TOOLS`) | Create a `TileMapLayer` node in the edited scene; optional parent and name. |
| `tilemap-set-tileset` | editor (`#if TOOLS`) | Assign a `TileSet` resource (by `res://` path) to a `TileMapLayer`. |
| `tilemap-set-cell` | editor (`#if TOOLS`) | Set a cell at coords `(x, y)` from a tileset `source_id` + atlas coords `(x, y)`. |
| `tilemap-erase-cell` | editor (`#if TOOLS`) | Erase the cell at coords `(x, y)`. |
| `tilemap-get-used-cells` | editor (`#if TOOLS`, read-only) | List the coordinates of all used (non-empty) cells. |
| `tilemap-clear` | editor (`#if TOOLS`) | Clear all cells in a `TileMapLayer`. |

Editor-driving tools live under `src/Godot-AI-Tilemap/Editor/` behind `#if TOOLS` and marshal every
Godot call onto the editor main thread via `MainThread.Instance.Run(...)`. Their pure-managed cores
(cell-coordinate / atlas-coordinate parsing and validation) and the tool-id constants live under
`Runtime/` outside `#if TOOLS` and are CI-unit-tested without a Godot binary.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon. Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.Tilemap`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.Tilemap" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary needed)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless:

```bash
dotnet build src/Godot-AI-Tilemap/Godot-AI-Tilemap.csproj            # compiles tools (Godot API resolves)
dotnet test  tests/Godot-AI-Tilemap.Tests/Godot-AI-Tilemap.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-Tilemap/Godot-AI-Tilemap.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Tilemap-Testbed.csproj                          # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon, and (once a local MCP server is wired) calls each tool via `godot-cli run-tool`.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes.

## Publish

Source-only, version-gated release (see `docs/ci.md`): publishing uses NuGet Trusted Publishing (OIDC,
no stored API key), bump `<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`;
`release.yml` runs the full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0**.
