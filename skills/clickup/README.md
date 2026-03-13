# ClickUp Skill

这个 skill 不再调用仓库内的脚本，而是直接调用全局安装的 `clickup` CLI。

如果你以前用的是旧入口：

```bash
node .agents/skills/clickup/query.mjs ...
```

现在统一改为：

```bash
clickup ...
```

适用场景：

- 查询 ClickUp task、comment、doc、page
- 创建和更新 task
- 搜索任务、查看我的任务、批注、修改状态
- 创建和编辑 ClickUp 文档页面

如果你是在维护 skill，本目录里的 [`SKILL.md`](./SKILL.md) 是给模型看的执行规范；本文档是给人看的安装、配置和命令说明。

## 安装全局 CLI

优先使用 npm 全局安装已发布包：

```bash
npm install -g @discountry/clickup-cli
```

如果你是在当前仓库本地联调，也可以在仓库根目录安装当前代码：

```bash
npm install -g .
```

安装完成后先验证命令是否可用：

```bash
clickup --help
```

## 环境变量配置

这个 skill 不依赖本地 `.env` 文件。CLI 直接读取 shell 环境变量。

当前 shell 临时生效的示例：

```bash
export CLICKUP_API_TOKEN=pk_your_token_here
export CLICKUP_WORKSPACE_ID=123456
export CLICKUP_USER_ID=7890
export CLICKUP_DEFAULT_LIST_ID=901111220963
```

如果你用的是 `zsh` 或 `bash`，想长期生效，就把这些 `export` 写到自己的 shell 配置文件里，例如 `~/.zshrc` 或 `~/.bashrc`，然后重新加载 shell。

变量说明：

| 变量名 | 是否必需 | 说明 |
| --- | --- | --- |
| `CLICKUP_API_TOKEN` | 必需 | ClickUp Personal API Token，通常以 `pk_` 开头 |
| `CLICKUP_WORKSPACE_ID` | 推荐 | Workspace ID；未设置时 CLI 会尝试回退到 API 返回的第一个 workspace |
| `CLICKUP_TEAM_ID` | 可选 | 旧字段别名，可替代 `CLICKUP_WORKSPACE_ID` |
| `CLICKUP_USER_ID` | 可选 | 当前用户 ID；不设置时 CLI 会自动探测 |
| `CLICKUP_DEFAULT_LIST_ID` | 可选 | 默认 List ID；配置后可以直接执行 `clickup create "Task title"` |

获取 token 的常见路径：

1. 打开 ClickUp。
2. 进入 `Settings`。
3. 打开 `Apps` 或 API Token 页面。
4. 创建或复制 Personal API Token。

注意事项：

- 不要把 token 写进仓库文件。
- 不要提交、打印或截图暴露真实 token。
- 这个 skill 约定从 shell 环境读取配置，不要求也不建议在 skill 目录里放 `.env`。

## 常用命令

### 任务与评论

```bash
clickup me
clickup get 86a1b2c3d
clickup get "https://app.clickup.com/t/86a1b2c3d" --subtasks
clickup comments 86a1b2c3d
clickup comment 86a1b2c3d "Starting work"
clickup status 86a1b2c3d
clickup status 86a1b2c3d "in progress"
clickup status 86a1b2c3d "progress"
clickup tasks 901111220963
clickup tasks 901111220963 --me
clickup my-tasks
clickup search "authentication"
```

### 创建和更新任务

```bash
clickup create "Quick task"
clickup create 901111220963 "New feature" --assignee me --due tomorrow --description "## Scope"
clickup assign 86a1b2c3d jane
clickup assign 86a1b2c3d jane@example.com
clickup assign 86a1b2c3d 123456
clickup due 86a1b2c3d "+3d"
clickup due 86a1b2c3d clear
clickup due 86a1b2c3d "2026-03-20"
clickup priority 86a1b2c3d high
clickup subtask 86a1b2c3d "Write tests"
clickup move 86a1b2c3d 901111220964
clickup link 86a1b2c3d "https://github.com/org/repo/pull/123" "PR #123"
clickup checklist 86a1b2c3d "Review code"
clickup delete-comment 90110200841741
clickup watch 86a1b2c3d alex
clickup watch 86a1b2c3d jane@example.com
clickup watch 86a1b2c3d 123456
clickup tag 86a1b2c3d "DevOps"
clickup description 86a1b2c3d "# Summary"
```

补充说明：

- `status <task>` 不带第二个参数时，会列出该任务当前可用状态。
- `status <task> "<text>"` 支持大小写不敏感和部分匹配，例如 `"progress"` 可以匹配 `"in progress"`。
- `assign` 和 `watch` 支持按用户名、邮箱、用户 ID 查询；部分情况下也可按部分用户名或邮箱匹配。
- `watch` 的实现是发送一条带 `@mention` 的评论，不是直接调用 ClickUp 的 watcher API。
- `priority` 支持 `urgent`、`high`、`normal`、`low`、`none`。
- `due` 可接受自然语言或相对时间，例如 `today`、`tomorrow`、`next friday`、`next week`、`+3d`。
- `due` 也支持显式日期，例如 `2026-03-20`；传 `none` 或 `clear` 可以清空截止时间。

### 文档与页面

```bash
clickup docs
clickup docs "API"
clickup doc abc123
clickup create-doc "Project Notes" --content "# Notes"
clickup page abc123 page456
clickup create-page abc123 "New Section" --content "Hello"
clickup edit-page abc123 page456 --name "Renamed"
clickup edit-page abc123 page456 --content "# Updated"
clickup edit-page abc123 page456 --name "Renamed" --content "# Updated"
```

补充说明：

- `edit-page` 至少需要 `--content` 或 `--name` 其中一个参数，两个都不传会直接报用法错误。

## 适合在 skill 中调用的方式

给模型或自动化流程使用时，优先遵循这些约定：

- 需要结构化字段时，加 `--json`
- 不确定对象当前状态时，先读再写
- 用户给的是完整 ClickUp URL 时，可以直接传给 `get`、`comments`、`status` 等命令
- 只在 `CLICKUP_DEFAULT_LIST_ID` 已配置时使用 `clickup create "Task title"` 这种简写
- 需要先判断状态名时，先运行一次 `clickup status <task>`
- 需要清空截止时间时，明确使用 `clickup due <task> clear`

示例：

```bash
clickup get 86a1b2c3d --json
clickup status 86a1b2c3d --json
clickup docs "roadmap" --json
clickup create 901111220963 "Prepare launch checklist" --assignee me --due "next friday"
```

## 故障排查

`clickup: command not found`

- 说明 CLI 还没全局安装，先执行安装步骤。

认证失败或 401

- 优先检查 `CLICKUP_API_TOKEN` 是否已导出到当前 shell。
- 确认 token 仍然有效，没有复制错账号或工作区。

找不到 workspace 或 list

- 检查 `CLICKUP_WORKSPACE_ID` 和 `CLICKUP_DEFAULT_LIST_ID`。
- 如果你依赖自动探测 workspace，建议改成显式配置，避免拿到错误 workspace。

创建任务时报 list 缺失

- 说明你用了 `clickup create "Task title"` 简写，但没有配置 `CLICKUP_DEFAULT_LIST_ID`。
- 改成显式传入 list ID，或者先配置默认 list。

## 对应 skill 文件

- [`SKILL.md`](./SKILL.md)：模型执行指令，描述何时触发、如何选择命令、如何处理常见错误
