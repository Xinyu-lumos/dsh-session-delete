# dsh-session-manager

[English](README.en.md) | 中文

<p align="center">
  <a href="./LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-3167E3?style=flat-square"></a>
  <img alt="DeepSeek Harness" src="https://img.shields.io/badge/DeepSeek%20Harness-0.1.0--rc.6-3167E3?style=flat-square">
  <img alt="Version" src="https://img.shields.io/badge/dsh--session--manager-v0.1.8-3167E3?style=flat-square">
</p>

`dsh-session-manager` 是一个功能全面的 DSH 会话管理插件，提供完整的会话管理功能：删除（含回收站/恢复/彻底清除）、恢复已归档会话、近期活动统计、继续/暂停会话、打开日志目录、未读/已读标记、新聊天中继续（fork）、工作区分组与排序管理、上下文压缩阈值设置。

**简洁版本**：移除了对话页面顶部的按钮入口，所有会话管理功能统一在设置中访问，保持界面清爽。

<sub><span style="opacity:.6">本项目基于 dsh-session-manager 修改，由 Xinyu-lumos 维护</span></sub>

<sub><span style="opacity:.6">如果觉得有用，欢迎点个 ⭐ Star，谢谢支持！</span></sub>

## 功能

- 设置页新增独立的「会话管理」分栏（与 Notifications 同级的设置分区）
- 面板列出全部会话（标题 / 工作目录），底部折叠区单独展示**已归档会话**，支持**一键恢复**回到会话列表
- **回收站**：删除的会话移入回收站（保留最近 10 条，超出自动清除最早一条），可**恢复**或**彻底删除**
- **统计**：每个会话可展开查看近期活动统计（轮次 / 用户消息 / 助手消息 / 工具调用 / 活动窗口）
- **继续会话**：一键打开会话并关闭面板；**暂停**：停止正在运行会话的当前回合
- **未读 / 已读**：会话行标题旁显示状态点——手动未读为蓝色、官方等待输入为琥珀、官方完成提醒为绿色、运行中为转圈；点击官方状态点**就地已读**（不跳转），点击蓝色点清除未读，打开会话自动已读；官方侧边栏的对应会话行旁同步显示蓝色未读点
- **新聊天中继续**：每个会话一键 fork 子会话（官方 `sessions.fork`）并打开
- **文件夹**：在系统文件管理器中打开会话日志目录
- **工作区管理**：会话按工作区分组展示，组内按最后使用时间排序（可切换最新/最旧）；拖拽工作区标题即可调整顺序（插入 / 交换 / 拖到末尾）；悬停标题出现**置于顶部 / 重命名 / 删除**按钮（删除按官方定义：仅移出列表，文件夹与会话记录保留，会话归入「未分组」）
- **上下文压缩阈值**（通用设置）：设置对话上下文用到模型窗口（100 万 token）的多少比例时自动压缩（17%–90%），每次压缩保留最近 16% 原文；**对所有 Agent 预设的会话统一生效**（保存即时 + 持久化 + 重启自动应用）
- 删除限制：仅禁止删除「正在思考」的会话；当前打开的会话（空闲）可删除
- 子代理（subagent）会话支持删除（非运行中）：即使主会话已删除、子代理成为「孤儿」，也能在会话管理中直接清理
- 中英文界面自适应（跟随页面语言）
- **简洁界面**：移除对话页面顶部的所有会话管理按钮，统一通过设置访问，保持对话界面清爽

## 与原版的区别

相比原版 [dsh-session-manager](https://github.com/dream12347/dsh-session-manager)，本版本：

✅ **移除了对话页面顶部的按钮**：
- ❌ 删除「删除本对话」按钮
- ❌ 删除「对话管理」按钮
- ❌ 删除「回收站」按钮

✅ **保留所有核心功能**：
- ✅ 设置页完整的会话管理
- ✅ 回收站（恢复/彻底删除）
- ✅ 已归档会话管理
- ✅ 活动统计
- ✅ 继续/暂停会话
- ✅ 未读/已读标记
- ✅ Fork 子会话
- ✅ 打开日志目录
- ✅ 工作区管理
- ✅ 上下文压缩阈值

## 截图

设置页「会话管理」分栏（工作区分组、行操作与回收站）：

![设置页会话管理](assets/settings-section.png)

通用设置「上下文压缩阈值」（17%–90%，滑块刻度）：

![上下文压缩阈值](assets/general-settings.png)

## 安装

### 从 GitHub

```sh
dsh plugin --profile web add 'https://github.com/Xinyu-lumos/dsh-session-manager.git'
```

### 从本地目录

```sh
dsh plugin --profile web add /absolute/path/to/dsh-session-manager
```

### 从 tarball

```sh
pnpm pack
dsh plugin --profile web add /absolute/path/to/dsh-session-manager-0.1.8.tgz
```

安装完成后**重启** `dsh web`（host 插件与客户端 bundle 需要重启加载）。

## 使用

### 设置页会话管理

1. 打开侧边栏底部 **设置**（齿轮图标）
2. 设置页面左侧导航出现独立的 **会话管理** 分栏，点击进入
3. 主列表为未归档会话；底部「已归档会话」折叠区可展开查看、**恢复**或删除归档会话
4. 删除会话 → 进入底部「回收站」折叠区（保留最近 10 条）
5. 回收站内可 **恢复**（回到会话列表）或 **彻底删除**（永久清除，不可恢复）
6. 每行操作：**继续会话**（打开并进入对话）、**暂停**（停止正在运行的回合）、**统计**（展开近期活动）、**文件夹**（打开日志目录）、**删除**
7. 工作区标题右侧（悬停显示）：**置于顶部**（挪到最前）、**重命名**、**删除**（红色，二次确认）
8. 拖拽工作区标题可调整顺序：放到某个工作区上方/下方插入，放到标题上交换位置，拖到最下方即移到末尾
9. 排序按钮（最新在前 / 最旧在前）切换组内会话的排列顺序

### 通用设置：上下文压缩阈值

1. 打开 **设置** → **通用设置**（General）
2. 找到「上下文压缩阈值」：滑块 / 输入框设置 17%–90%
3. 保存后立即生效（含已打开的会话）；配置对所有 Agent 预设统一生效，重启后依然有效

### 未读 / 已读状态点

会话行标题旁的圆点表示四种状态：**蓝色**=手动标记未读、**琥珀**=官方等待输入、**绿色**=官方完成提醒、**转圈**=运行中。点击琥珀 / 绿色点**就地标记已读**（不跳转，仅清除官方提醒）；点击蓝色点清除未读；点击空白位置标记未读；打开会话自动已读。官方侧边栏的会话行旁同步显示蓝色未读点（按标题文本匹配，重复标题会话会共享该点）。

## 工作原理

| 层 | 实现 |
|---|---|
| Host | `src/index.ts` 注册 7 条路由：`POST /delete`（归档 + 非 live 会话文件移入回收站 + 记录条目）、`POST /restore`（文件移回 + 取消归档 + 删除条目）、`POST /purge`（清除回收站与原位置文件 + 删除条目）、`GET /trash`（回收站列表）、`POST /open-folder`（打开日志目录）、`POST /pause`（暂停运行中会话）、`GET|POST /compaction-threshold`（读写全局压缩阈值）。通过 `ctx.sessionPersistence` 定位会话、`ctx.workspaceRegistry` 归档/取消归档、`ctx.storageDomain` 持久化回收站条目、归档集合与阈值；`ctx.agents` 检测运行中的会话并拒绝删除 |
| Client | `src/client/index.ts` 通过官方 `settings.section` 插槽注册独立分栏，用 `useSessions` / `useWorkspaces` 标准数据源列出会话（含归档/回收站分组），删除/恢复/彻底删除调用 host 路由；抽屉通过 `sessions.list`（ObservableSnapshot）订阅实时列表；彻底删除的会话 id 记录在浏览器 localStorage，避免 live 会话删除后刷新「复活」；**移除了 `conversation.session.header.utilities` 插槽注册**，不在对话顶部显示任何按钮 |

- **未读机制**：手动未读集保存在浏览器 localStorage 的共享 key `dsh.session-unread.v1`（`{version:1, ids:[]}` 格式，与其他会话管理插件互通）；官方状态点（琥珀/绿色/转圈）由官方 `SessionSummary` 的 `pendingInteraction` / `completed` / `running` 字段驱动，点击就地已读通过清除官方提醒标记实现，无需打开会话；侧边栏的蓝色未读点由 MutationObserver 装饰官方树节点（官方行元素没有会话 id 属性，故按标题文本匹配）
- **压缩阈值全局生效**：保存在存储域（`dsh_delete_session` 的 `thresholdRatio`）与当前 Agent 预设的 `agent.cordis.yml`；host 在每个 `agent/pre-step` 钩子里把阈值写入所有预设的压缩引擎配置，因此对所有 Agent 预设的会话统一生效，重启后依然有效
- 删除时先走官方归档通道：侧边栏立即隐藏该会话
- 回收站条目持久化在 DSH 存储域（`~/.dsh/storages/dsh_delete_session.json`），文件在 `~/.dsh/dsh-delete-session-trash/`
- 工作区记账（`sessionIds` 槽位）在下次启动时由 registry 重建索引自动对账，无需手动编辑文件
- 无系统提示词改动、无模型工具新增，对 token 与模型行为零影响

## 限制

- **不能删除正在运行的会话**（按钮禁用并拒绝删除），多标签页场景请先在别处确认该会话已停止
- **子代理会话可删除**（非运行中），包括主会话已删除的「孤儿子代理」——不会再有无法清理的残留会话
- live 会话（当前进程内打开的会话）删除后，其内存状态由 DSH 在重启时彻底清理
- 已彻底删除的会话 id 会保留在浏览器 localStorage（防止刷新后重新出现）与归档集合中（无害残留，不显示）
- 侧边栏未读点按标题文本匹配，存在重复标题的会话时两者会共享同一个点（会话管理抽屉内不受影响，按真实会话 id 精确标记）

## 兼容性

当前版本适配 DSH `0.1.0-rc.6`（依赖 `settings.section` / `settings.general.item` 插槽与 `ctx.sessionPersistence` / `ctx.workspaceRegistry` / `ctx.agents` / `ctx.storageDomain` / `ctx.agentPresets` 服务）。DSH 版本升级后如插槽或服务 API 变化，需要同步适配。

## 开发

```sh
pnpm install        # 安装依赖（@deepseek-ai 系列为 link 本地开发依赖）
pnpm run check      # typecheck + test + build
```

`lib/` 为提交的构建产物，修改源码后必须重新构建并提交 `lib/`。

## 致谢

基于 [dsh-session-manager](https://github.com/dream12347/dsh-session-manager) 修改，移除对话页面顶部的按钮入口，提供更简洁的界面体验。

## License

MIT