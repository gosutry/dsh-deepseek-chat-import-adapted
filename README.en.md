# dsh-deepseek-chat-import-adapted

Back-ports the official DeepSeek Harness experimental plugin **DeepSeek chat import** (built in from `0.2.2-rc.3` onward) to earlier runtimes: **`>=0.2.1-rc.3 <0.2.2-rc.3`**.

**The repository root is the plugin package itself**: `package.json`, `lib/`, `locale/` and `cordis.patch.yml` all sit at the top level.
Clone it and wire it up with the steps below; no build step and no npm dependency install are required.

English | [中文](README.md)

## Layout

| Path | Purpose |
|---|---|
| `package.json` `lib/` `locale/` `icon.svg` `cordis.patch.yml` | The plugin package (only `lib/` is loaded at runtime) |

## Provenance

| Item | Value |
|---|---|
| Upstream repository | [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) |
| Upstream tag | [`dsh-v0.2.2-rc.3`](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.2-rc.3) |
| Introducing commit | [`b8c33a150`](https://github.com/deepseek-ai/deepseek-harness/commit/b8c33a150) |
| Merged PR | [#8342 "import DeepSeek website chat exports as Sessions"](https://github.com/deepseek-ai/deepseek-harness/pull/8342) |
| Upstream directory | [`packages/experimental/deepseek-chat-import`](https://github.com/deepseek-ai/deepseek-harness/tree/dsh-v0.2.2-rc.3/packages/experimental/deepseek-chat-import) |
| Upstream package name | [`@deepseek-ai/dsh-deepseek-chat-import`](https://www.npmjs.com/package/@deepseek-ai/dsh-deepseek-chat-import) · [registry metadata](https://registry.npmjs.org/@deepseek-ai/dsh-deepseek-chat-import) |
| License | [MIT](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.3/LICENSE) (local copy in [LICENSE](LICENSE)) |

> The package keeps its official name `@deepseek-ai/dsh-deepseek-chat-import`: it is an official package, and keeping the name
> preserves provenance and keeps it consistent with the upstream directory, `cordis.patch.yml` and the references in a profile. This repository is a derived copy, not a new package.

This repository does not fork the upstream monorepo (~380 MB, 59 subdirectories); it carries only this package plus the adaptation notes.

## Status

- Supported range: DSH `>=0.2.1-rc.3 <0.2.2-rc.3`
  - Lower bound: the sidebar extension point `sidebar.panellist.actions` first appears in `0.2.1-rc.3` (zero hits in rc.1/rc.2, so the entry would be invisible)
  - Upper bound: `0.2.2-rc.3` and later ship this plugin natively; no installation needed
  - Runtime verification: only `0.2.2-rc.1` was actually run; for `0.2.1-rc.3`–`rc.5` and `0.2.2-rc.2` only the required symbols were checked — **not run**
- Code source: upstream 0.2.2-rc.3 (`packages/experimental/deepseek-chat-import`)
- Package version: `0.2.1-rc.3-adapted` (matches the Release tag; the supported range is expressed by `peerDependencies`)
- Verified on `0.2.2-rc.1`: the entry appears in the sidebar **More** menu; the dialog, the Workspace dropdown and a real `deepseek_data-*.zip` import all work
- `lib/client.js` sha256: `51b1048253c32c46b34bd83696c630c12d87ea6cb31b0b511cf343b251624de2`
- Release asset: `deepseek-ai-dsh-deepseek-chat-import-0.2.1-rc.3-adapted.tgz`, sha256 `b41934715fa3546624a24b1b5d7a80bbc5ab2d67748cc46d938dfe17f5ec3f6f`

## Installation

No fixed paths anywhere: you choose where the plugin lives, `dsh` resolves the profile itself,
and the host dependency tree comes from `npm root -g`. Below, `<plugin-dir>` is wherever you put this repository.

1. Register it in a profile (this writes the dependency and appends the bundle to `dsh.profile.bundles`):

```bash
dsh plugin --profile web link "<plugin-dir>"
```

2. Let the plugin resolve the host packages (DSH provides these; do not install them from npm again).
Create a `node_modules` inside the plugin directory pointing at the DSH dependency tree:

**macOS / Linux / Android**

```bash
ln -sfn "$(npm root -g)/@deepseek-ai/dsh/node_modules" "<plugin-dir>/node_modules"
```

**Windows (cmd; administrator or Developer Mode)**

```bat
rem inside a .bat file write %%i instead of %i
for /f "delims=" %i in ('npm root -g') do mklink /J "<plugin-dir>\node_modules" "%i\@deepseek-ai\dsh\node_modules"
```

**Windows (PowerShell)**

```powershell
$root = (npm root -g).Trim()
New-Item -ItemType Junction -Path "<plugin-dir>\node_modules" -Target "$root\@deepseek-ai\dsh\node_modules"
```

3. Restart the DSH runtime afterwards (the profile sets `patchReload: startup`, and client bundles are pinned by a hash computed at boot).

Entry point: sidebar → **More** → first row "导入 DeepSeek 会话" (Import DeepSeek chats).

### Alternative: edit the profile by hand

Instead of the command, edit that profile's `package.json` (relative paths are resolved against that file,
so they can point at wherever the plugin lives):

```json
"dependencies": {
  "@deepseek-ai/dsh-deepseek-chat-import": "link:../../plugin-sources/dsh-deepseek-chat-import-adapted"
},
"dsh": { "profile": { "bundles": [ ..., "@deepseek-ai/dsh-deepseek-chat-import" ] } }
```

> With `link:` pnpm warns that peer dependencies of the target are not resolved. That is expected —
> the dependency link from step 2 is what makes host packages such as `@deepseek-ai/dsh-session` resolvable.
