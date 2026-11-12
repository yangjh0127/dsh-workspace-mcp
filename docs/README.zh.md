# DSH Workspace MCP

[English](../README.md) | 简体中文

DSH Workspace MCP 从工作区配置加载 MCP 服务器，并按会话所属的工作区隔离工具。插件运行期间，已发现的工作区保持挂载，切换工作区时现有连接继续保留。

需要 DeepSeek Harness `0.2` 系列，最低版本为 `0.2.2-rc.1`，以及 Git、Node.js 和 pnpm。[下载 DeepSeek Harness](https://www.deepseek.com/harness/)

## 安装

克隆本仓库，构建后链接到你使用的 DSH profile。请把仓库放在固定目录，DSH 会从该目录加载编译产物。

```powershell
git clone https://github.com/yangjh0127/dsh-workspace-mcp.git
cd dsh-workspace-mcp
pnpm install --frozen-lockfile
pnpm run build
dsh plugin --profile web add "link:$((Get-Location).Path -replace '\\','/')"
```

根包自带 MCP 管理页。按实际使用的 profile 选择 `web` 或 `desktop`，安装后重启 DSH。更新时，在已链接的仓库目录依次运行 `git pull --ff-only`、`pnpm install --frozen-lockfile` 和 `pnpm run build`，再重启 DSH。

## 配置 MCP 服务器

- 在工作区根目录使用 `.dsh/mcp.yml` 或 `.dsh/mcp.json`。插件也兼容读取工作区的 `.mcp.json`。配置变更会自动重载。
- 从主页侧栏、设置侧栏或 **设置 → 内置插件** 打开 **MCP 管理**。可在 **设置 → 通用 → MCP 管理页入口** 选择显示哪些入口。Headless profile 可使用装载器和命令行。
- 默认使用 **Eager（立即加载）**，发现工作区后即挂载服务器。也可为单台服务器选择 **Lazy（按需加载）**，使用 `mcp_search` 查找工具、`mcp_describe` 查看参数结构、`mcp_call` 执行调用。连接保持至会话结束或服务器条目卸载。
- 工作区工具按会话原始工作区隔离。用户层和 profile 层可以提供跨工作区共享的服务器。

`stdio` 条目会在 DSH 宿主上执行配置中的命令。请只启用可信的服务器和工作区配置。

## 文档

- [配置格式](guide/format.zh.md)
- [配置来源和优先级](guide/layers.zh.md)
- [环境变量展开](guide/env-expansion.zh.md)
- [命令行参考](guide/cli.zh.md)
- [插件服务接口](guide/service.zh.md)

运行 `pnpm test` 可构建并测试插件。问题请提交到本仓库的 [Issues](https://github.com/yangjh0127/dsh-workspace-mcp/issues)，PR 请以 `dev` 为目标分支。

## 许可证

[MIT](../LICENSE)
