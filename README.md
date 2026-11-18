# dsh-deepseek-chat-import-adapted

把 DeepSeek Harness 官方实验插件 **DeepSeek 会话导入**（rc.3 内置）适配到 **0.2.2-rc.1** 运行时。

**本仓库根目录就是插件本体**：`package.json`、`lib/`、`locale/`、`cordis.patch.yml` 都在顶层，
clone 下来按下面的安装方法接入即可，不需要构建、不需要装 npm 依赖。

## 目录

| 路径 | 作用 |
|---|---|
| `package.json` `lib/` `locale/` `icon.svg` `cordis.patch.yml` | 插件本体（运行时只加载 `lib/`） |

## 来源

| 项 | 值 |
|---|---|
| 上游仓库 | [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) |
| 上游 tag | [`dsh-v0.2.2-rc.3`](https://github.com/deepseek-ai/deepseek-harness/releases/tag/dsh-v0.2.2-rc.3) |
| 引入提交 | [`b8c33a150`](https://github.com/deepseek-ai/deepseek-harness/commit/b8c33a150) |
| 合并 PR | [#8342「导入 DeepSeek 官网对话导出为会话」](https://github.com/deepseek-ai/deepseek-harness/pull/8342) |
| 上游目录 | [`packages/experimental/deepseek-chat-import`](https://github.com/deepseek-ai/deepseek-harness/tree/dsh-v0.2.2-rc.3/packages/experimental/deepseek-chat-import) |
| 上游包名 | [`@deepseek-ai/dsh-deepseek-chat-import`](https://www.npmjs.com/package/@deepseek-ai/dsh-deepseek-chat-import) · [registry 元数据](https://registry.npmjs.org/@deepseek-ai/dsh-deepseek-chat-import) |
| 许可证 | [MIT](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.2-rc.3/LICENSE)（副本见 [LICENSE](LICENSE)） |

> 包名保持官方原名 `@deepseek-ai/dsh-deepseek-chat-import`：它本来就是官方包，保留原名既能保留来源，
> 也与上游目录、`cordis.patch.yml` 以及 profile 中的引用保持一致。本仓库是派生副本，不是另起的包。

本仓库不 fork 上游 monorepo（约 380 MB、59 个子目录），只保留这一个包 + 适配记录。

## 状态

- 适配范围：DSH `>=0.2.1-rc.3 <0.2.2-rc.3`
  - 边界依据：`sidebar.panellist.actions` 这个侧栏扩展点首次出现于 `0.2.1-rc.3`（rc.1/rc.2 命中 0，装了也看不到入口）
  - 上限：`0.2.2-rc.3` 起官方已内置该插件，无需安装
  - 实测运行：仅在 `0.2.2-rc.1` 上真跑过；`0.2.1-rc.3`~`rc.5`、`0.2.2-rc.2` 只核对过依赖符号齐全，**未实测运行**
- 代码来源：上游 0.2.2-rc.3（`packages/experimental/deepseek-chat-import`）
- 包版本：`0.2.2-adapted`（版本号只表示「基于 0.2.2 系列的适配版」，支持范围以 peerDependencies 为准）
- 在 `0.2.2-rc.1` 上实测：入口出现在侧栏「更多」菜单，弹窗、工作区下拉可用，真实 `deepseek_data-*.zip` 导入成功
- `lib/client.js` sha256：`51b1048253c32c46b34bd83696c630c12d87ea6cb31b0b511cf343b251624de2`

## 安装

全程不依赖固定路径：插件放哪由你决定，profile 位置由 `dsh` 自己解析，
宿主依赖位置由 `npm root -g` 推导。下文 `<插件目录>` 指你放置本仓库的目录。

1. 安装到 profile（会自动写入依赖并把 bundle 加进 `dsh.profile.bundles`）：

```bash
dsh plugin --profile web link "<插件目录>"
```

2. 让插件能解析宿主依赖（这些包由 DSH 提供，不要从 npm 再装一份）。
插件目录下建一个指向 DSH 自身依赖树的 `node_modules`：

**macOS / Linux / Android**

```bash
ln -sfn "$(npm root -g)/@deepseek-ai/dsh/node_modules" "<插件目录>/node_modules"
```

**Windows（cmd，管理员或已开启开发者模式）**

```bat
rem .bat 文件内把 %i 写成 %%i
for /f "delims=" %i in ('npm root -g') do mklink /J "<插件目录>\node_modules" "%i\@deepseek-ai\dsh\node_modules"
```

**Windows（PowerShell）**

```powershell
$root = (npm root -g).Trim()
New-Item -ItemType Junction -Path "<插件目录>\node_modules" -Target "$root\@deepseek-ai\dsh\node_modules"
```

3. 装完重启 DSH 运行时（profile 的 `patchReload` 是 `startup`，客户端包也按启动哈希固化）。

入口位置：左侧栏 → **更多** → 第一行「导入 DeepSeek 会话」。

### 备选：手写 profile

不想用命令，也可以直接改该 profile 的 `package.json`（相对路径以它自身为基准，
指向插件所在的任意目录）：

```json
"dependencies": {
  "@deepseek-ai/dsh-deepseek-chat-import": "link:../../plugin-sources/dsh-deepseek-chat-import-adapted"
},
"dsh": { "profile": { "bundles": [ ..., "@deepseek-ai/dsh-deepseek-chat-import" ] } }
```

> `link:` 安装时 pnpm 会提示“不解析目标的 peerDependencies”，属预期行为——
> 第 2 步的依赖链接才是让 `@deepseek-ai/dsh-session` 这类宿主包可解析的关键。
