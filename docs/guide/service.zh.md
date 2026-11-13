# 查询面（`projectMcp`）

[English](service.md) | 中文

[← 返回 README](../README.zh.md) ｜ 相关：[配置来源与分层](layers.zh.md) · [CLI `dsh-mcp`](cli.zh.md) · [设置页](settings-ui.zh.md) · [Lazy 设计记录](../design/lazy-mcp.md)

其它插件和配套 UI 经 cordis 服务 `ctx.projectMcp` 读装载状态。核心服务只在进程内
发事件：对账成功结束时通知一次，装载在那之后才完成时再通知一次。浏览器走 bundle
内的配套 UI 自己建的 SSE，核心服务不打开那条通道。

## 方法

| 方法 | 行为 |
|---|---|
| `snapshot()` | 上一轮对账的内存分区（项目文件与用户层）。进对账队列，不读盘，不发起对账。 |
| `serverView(projectRoot, rawName)` | 一行的内存 view。排队规则与 `snapshot()` 相同。 |
| `globalState(rawName)` | 该原名的全局层运行态；项目行返回 `undefined`。 |
| `reload()` | 一次全量对账（`reconcileNow`）。 |
| `writeTargets()` | 添加对话框当前能写的受管 yml：当前工作区、用户层、当前 profile。 |
| `addServer(source, projectRoot, draft)` | 按表单追加一条受管 yml 行（没有文件就创建）并排一次对账。同名已在该文件里则拒绝，文件不动。 |
| `setServerEnabled(source, projectRoot, rawName, enabled)` | 受管 yml 行就地翻 `disabled`。其它来源的行在同作用域受管 yml 里落一条：启用写完整拷贝，停用写 `disabled: true` 占位。 |
| `setLoadingMode(source, projectRoot, rawName, mode)` | 把 `mode`（`eager`/`lazy`）写进该作用域的受管 yml 行。其它来源的行在受管 yml 还没有对应行时完整复制过去；原 JSON / `.mcp.json` 不动；`disabled` 保持原值。缺少有效连接配置的占位行会被拒绝，文件不动。写完排一次对账并返回文件路径，**不等待对账结束**。 |
| `removeServer(source, projectRoot, rawName)` | 删掉受管 yml 行；低层还有同名或同服务身份时改留 `disabled: true` 占位。其它来源的行落一条占位。 |
| `setToolEnabled(source, projectRoot, rawName, tool, enabled)` | 把一个精确工具名写进受管行的 `tools.allow` / `tools.deny`。含 glob 字符的拒绝。 |
| `toolStates(projectRoot, rawName)` | 已注册工具（短名）与各自是否可见。服务器未 active 时为空。 |
| `managedPathFor(source, projectRoot)` | 该来源的行会写到哪份受管 yml；解析不出 profile 名时为 `undefined`。 |
| `prepareManagedYml(source, projectRoot)` | 受管 yml 不存在时建一个 YAML 空列表（`[]`），返回绝对路径；第一次写服务器时再生成受管块。 |
| `openConfigFile(source, projectRoot)` | 经宿主注入的 opener 打开受管 yml。 |
| `subscribeUpdated(listener)` | 订阅下文的更新信号；返回退订函数。 |

服务器与工具修改方法（`addServer`、`setServerEnabled`、`setLoadingMode`、
`removeServer`、`setToolEnabled`）只碰三份受管 yml（工作区 `.dsh/mcp.yml`、
`~/.dsh/mcp.yml`、`~/.dsh/profiles/<当前>/mcp.yml`），会校验允许的写入位置；非焦点
工作区在改文件前拒绝。`setLoadingMode` 还会先把选中的行重新按服务器输入解析；没有
有效连接配置的占位行在写盘前拒绝。每次修改都在写文件后排一次对账，返回时对账可能
尚未完成；要等对账结束再读结果，调用 `reload()`。路径查询、文件准备和打开接口不代表
相同的修改权限检查，也不触发对账。JSON 文件与 `.mcp.json` 从不写。

`snapshot()` / `serverView()` / `globalState()` 不发事件。

## 加载模式

`McpLoadingMode` 是 `"eager" | "lazy"`。`mode` 在所有结构里都是可选字段，省略等价于
`eager`：`McpServerView.mode` 只在行里显式写了 `lazy`/`eager` 时出现，
`McpAddDraft.mode` 由添加表单按需携带（只在选 Lazy 时发送），`snapshot()` /
`serverView()` 返回的视图会带上它。

`setLoadingMode()` 只写受管 yml：受管行已存在时只替换 `config.mode`；行来自 JSON /
`.mcp.json` 且受管 yml 还没有对应行时，把整行复制进受管 yml 再写 `mode`，原文件不动。
写盘前会重新按服务器输入解析选中的行；没有有效连接配置的占位行（例如只有 id 的手写
disabled 行）会被拒绝，文件保持不变。它不改 `disabled`：给停用行切模式，行仍然停用。
目标位置必须是 `writeTargets()` 里的当前工作区 / 用户层 / 当前 profile；其它工作区
在写文件前拒绝。

写文件后排一次 `reconcileAll`，但方法本身不等待对账结束；要等对账完成再读结果，调用
`reload()`。两种模式的运行差异、会话连接生命周期、搜索扇出和限制见
[配置格式](format.zh.md#加载模式) 与 [Lazy 设计记录](../design/lazy-mcp.md)。

## 事件

每次 **成功** 的 `reconcileAll` 结束都会 emit——指纹未变、因而跳过重读、但仍跑完
健康巡检、会话 deny、工具预算和摘要的那一轮也算。装载 fiber 如果在这次对账
**结束之后**才变成 active 或 failed，会再发一次：连接完成时 `fiberPhase` 和
`toolCount` 才落定，不补这一次的话，设置页会停在「正在启动」，直到关掉重开。
对账进行中 settle 的不另发，收尾那一次已经读得到新状态。注册表调用：

```ts
ctx.on("projectMcp/updated", async () => {
  const snapshot = await ctx.projectMcp.snapshot();
});
```

事件名是 `PROJECT_MCP_UPDATED_EVENT`（`"projectMcp/updated"`）。无载荷，也不带
diff。派发是同步的，不会等待监听方返回的 Promise。监听方读 `snapshot()` 或
`serverView()`，和自己上次推出去的结果比较。对账中途因 `disposed` 返回，或对账
内部抛错，都不发。监听方抛错会记一条 `项目 MCP 变更事件投递失败：…`，对账本身
仍算成功。

## 类型

具名视图类型从包入口导出。`exports` 不开放 `lib/registry.js` 深导入。

```ts
import {
  PROJECT_MCP_SERVICE,
  PROJECT_MCP_UPDATED_EVENT,
  type FiberPhaseView,
  type McpAddDraft,
  type McpLoadingMode,
  type McpRowSource,
  type McpScopeInfo,
  type McpServerRuntimeView,
  type McpServerView,
  type McpTransport,
  type PatchRow,
  type ProjectFileState,
  type ProjectMcpService,
  type ProjectServerPhase,
  type ProjectServerState,
  type ReconnectConfig,
  type ToolFilter,
} from "dsh-workspace-mcp";
```

`snapshot()` 的元素类型是 `ProjectFileState`。每一行是 `McpServerRuntimeView`
（`McpServerView` 再加 `source`、`fiberPhase`、`skipReason`、`toolCount`）。
`globalState()` 返回 `ProjectServerState`。`McpServerView.mode` 可选；缺省表示 Eager。
`McpAddDraft.mode` 同样可选，添加表单只在选 Lazy 时发送。

## 对配套 UI 的语义化版本

`projectMcp` 的方法、`projectMcp/updated`，以及上面导出的视图类型，**对配套 UI**
按语义化版本承诺：

| 版本位 | 对这个面意味着 |
|---|---|
| patch | 修复，方法、事件和视图形状不变。 |
| minor | 只增加字段、方法或事件。 |
| major | 删除、改名，或改变含义。 |

`setLoadingMode` 是新增方法，`McpLoadingMode`、`McpAddDraft.mode`、`McpServerView.mode`
是新增类型/字段，按上表都属于 **minor** additive 变更。根目录构建命令先编译装载器，再用当前的类型声明编译 UI。安装包同时包含两个组件。
