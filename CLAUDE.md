# CLAUDE.md — Godot-AI-Tilemap

A **Godot-MCP extension**: an MCP tool family for Godot's built-in `TileMapLayer` node (Godot 4.3+, the
modern replacement for the deprecated `TileMap`), shipped as a **source-only NuGet package**
(`com.IvanMurzak.Godot.MCP.Tilemap`) that compiles inside a consumer's Godot project against the
consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe
is the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

## Layout

- `src/Godot-AI-Tilemap/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_Tilemap.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_Tilemap.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tilemap/` — pure-managed support types: cell-coordinate / atlas-coordinate parsing and
    validation cores (all unit-tested, no Godot native API).
  - `Editor/Tools/Tool_Tilemap.{Editor,Create,SetTileset,SetCell,EraseCell,GetUsedCells,Clear}.cs` —
    editor tools behind `#if TOOLS` (touch `EditorInterface` / live `TileMapLayer` + `TileSet`;
    main-thread-marshalled; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.Tilemap.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-Tilemap.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary).
- `testbed/Tilemap-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `tilemap-create` | editor | `Editor/Tools/Tool_Tilemap.Create.cs` |
| `tilemap-set-tileset` | editor | `Editor/Tools/Tool_Tilemap.SetTileset.cs` |
| `tilemap-set-cell` | editor | `Editor/Tools/Tool_Tilemap.SetCell.cs` |
| `tilemap-erase-cell` | editor | `Editor/Tools/Tool_Tilemap.EraseCell.cs` |
| `tilemap-get-used-cells` | editor | `Editor/Tools/Tool_Tilemap.GetUsedCells.cs` |
| `tilemap-clear` | editor | `Editor/Tools/Tool_Tilemap.Clear.cs` |

## Build / test (no Godot binary)

```bash
dotnet build src/Godot-AI-Tilemap/Godot-AI-Tilemap.csproj   # source-only package compiles tools
dotnet test  tests/Godot-AI-Tilemap.Tests/Godot-AI-Tilemap.Tests.csproj
dotnet pack  src/Godot-AI-Tilemap/Godot-AI-Tilemap.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Tilemap-Testbed.csproj                 # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
The recipe is verified to compile into the consumer across the CI's multi-Godot matrix (4.3 / 4.4 / 4.5).
When proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an already-cached version:
if you re-pack the same `Version`, clear `~/.nuget/packages/com.ivanmurzak.godot.mcp.tilemap/<ver>` (or
pack a unique version) before re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.Tilemap`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed cores (no Godot native API) + the tool-id consts → `Runtime/` (outside `#if TOOLS`,
  unit-testable); editor-driving tools → `Editor/` (behind `#if TOOLS`, every Godot call via
  `MainThread.Instance.Run(...)`, E2E-verified). Godot types that wrap a native object (`Node`,
  `Resource`, `NodePath`, `SceneTree`) P/Invoke on construction, so they must never be touched from the
  no-Godot xUnit host.
- The package declares ONLY the `com.IvanMurzak.McpPlugin` / `com.IvanMurzak.ReflectorNet` min-version
  deps; **GodotSharp must never become a package dependency** (CI asserts the nuspec). Keep the MCP pins in
  lockstep with the core Godot-MCP addon; bump with `commands/update-core.ps1`.
- One `[AiToolType] partial class Tool_Tilemap`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, the NuGet trusted-publishing setup.
