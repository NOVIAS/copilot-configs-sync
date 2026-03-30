# GitHub Copilot Configs Sync Project

> 自动同步 Awesome GitHub Copilot 配置文件的 GitHub Actions 工作流项目

## 项目描述

该项目提供了一个完整的 GitHub Actions 工作流来自动同步 [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot) 仓库中的 Copilot 自定义配置文件，包括：

- **聊天模式 (chatmodes)** - GitHub Copilot 聊天模式配置
- **指令文件 (instructions)** - Copilot 自定义指令
- **提示文件 (prompts)** - 可复用的提示模板
- **代理文件 (agents)** - 自定义代理配置

## 核心功能

### 1. 自动化同步
- ✅ 每天北京时间 00:00 自动同步最新配置
- ✅ 支持手动触发同步
- ✅ 可指定上游源分支或tag

### 2. 安全的 PR 工作流
- ✅ 同步结果提交到独立同步分支 `automation/sync-copilot-configs`
- ✅ 自动创建或更新 PR，而不是直接推默认分支
- ✅ 支持默认分支受保护的仓库

### 3. 周归档分支
- ✅ 每周一北京时间 00:00 自动创建周分支
- ✅ 分支命名规则：`YYYYMM-第N周`（如 `202603-第5周`）
- ✅ 安全的分支刷新机制（force-with-lease）

## 项目结构

```
copilot-configs-sync/
├── .github/
│   └── workflows/
│       └── sync-copilot-configs.yml       # 主要 GitHub Actions workflow
├── docs/
│   ├── setup.md                           # 设置指南
│   ├── configuration.md                   # 配置说明
│   └── troubleshooting.md                 # 故障排除
├── README.md                              # 本文件
├── SYNC_README.md                         # 同步说明模板
└── .gitignore                             # Git 忽略设置
```

## 快速开始

### 1. Fork 本项目或复制 workflow 文件

将 `.github/workflows/sync-copilot-configs.yml` 文件复制到你的仓库中，保持目录结构不变。

### 2. 基本配置

该 workflow 已预配置默认值，无需额外配置即可使用：

- **同步触发时间**：每天北京时间 00:00（UTC 16:00）
- **同步源**：https://github.com/github/awesome-copilot （main 分支）
- **同步分支**：`automation/sync-copilot-configs`

### 3. 推送到 GitHub

```bash
git add .
git commit -m "chore: add copilot configs sync workflow"
git push origin main  # 或你的默认分支
```

### 4. 启用 Actions

确保仓库的 **Settings > Actions > General** 中已启用 GitHub Actions。

## 工作流详解

### 自动同步流程

```
每天北京时间 00:00
    ↓
[1] Checkout 仓库（完整历史）
    ↓
[2] 检测执行事件（定时/手动）& 北京时间判断
    ↓
[3] 克隆 awesome-copilot（稀疏检出）
    ↓
[4] 同步 4 个目录到 .github/
    ├── .github/chatmodes/
    ├── .github/instructions/
    ├── .github/prompts/
    └── .github/agents/
    ↓
[5] 更新 SYNC_README.md & 生成提交
    ↓
[6] 推送至 automation/sync-copilot-configs 分支
    ↓
[7] 创建 PR（如有内容变化）
    ↓
[8] 周一时创建周归档分支（如已启用）
```

## 手动触发

在 GitHub Actions 页面手动运行 workflow，支持以下输入：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `source_ref` | String | `main` | awesome-copilot 源分支/tag/commit SHA |
| `create_weekly_branch` | Boolean | `false` | 强制创建本周归档分支 |

## 权限配置

工作流需要的最小权限：

```yaml
permissions:
  contents: write        # 允许推送分支和提交
  pull-requests: write   # 允许创建和编辑 PR
```

## 环境变量

所有关键变量已在 workflow 中定义，可按需调整：

| 变量 | 值 | 说明 |
|------|-----|------|
| `DEFAULT_SOURCE_REF` | `main` | 默认源分支 |
| `SOURCE_REPO` | `https://github.com/github/awesome-copilot.git` | 源仓库 URL |
| `SYNC_BRANCH` | `automation/sync-copilot-configs` | 同步分支名 |

## 周分支命名规则

周分支名按照"本月第几个周一"自动计算，不是周数。示例：

- 2026 年 3 月 2 日（第 1 个周一）→ `202603-第1周`
- 2026 年 3 月 9 日（第 2 个周一）→ `202603-第2周`
- 2026 年 3 月 16 日（第 3 个周一）→ `202603-第3周`
- 2026 年 4 月 6 日（第 1 个周一）→ `202604-第1周`

## 常见问题

### Q: Workflow 执行失败怎么办？

**A:** 检查以下几点：

1. GitHub Actions 已启用
2. GITHUB_TOKEN 有 `contents:write` 和 `pull-requests:write` 权限
3. 网络能访问 github.com
4. 日志中的具体错误信息

### Q: 如何修改同步时间？

**A:** 编辑 `.github/workflows/sync-copilot-configs.yml` 的 cron 表达式：

```yaml
schedule:
  - cron: '0 16 * * *'  # 改为你需要的时间（UTC）
```

北京时间 00:00 对应 UTC 16:00（前一天）。

### Q: 如何跳过某个星期的归档分支创建？

**A:** 周分支的创建完全自动化，但可以在本周一手动删除分支，下周一时会重新创建。

### Q: 怎样避免 workflow 冲突？

**A:** Workflow 已配置 `concurrency` 限制，不会有并发冲突。同时分支使用 `--force-with-lease` 确保安全。

## 依赖

- GitHub Runner 标准环境（无需额外依赖）
- `git` - 版本控制
- `rsync` - 目录同步
- `gh` - GitHub CLI（GitHub Runner 默认有）

## 工作流输出

每次运行 workflow 会生成：

1. **SYNC_README.md** - 更新的同步说明文档
2. **PR** - 同步内容变化时自动创建 PR
3. **周分支** - 每周一自动创建或刷新
4. **logs** - GitHub Actions 运行日志

## 调试技巧

### 查看完整日志

在 GitHub Actions 页面点击对应的 workflow run，展开各个 step 查看详细日志。

### 本地测试脚本逻辑

```bash
# 测试 cron 表达式有效性
TZ=Asia/Shanghai date +%u  # 输出 1 表示周一

# 测试周计算
current_day=$(TZ=Asia/Shanghai date +%d)
current_year_month=$(TZ=Asia/Shanghai date +%Y%m)
echo "当前日期信息: ${current_year_month}-${current_day}"
```

## 许可证

本项目示例代码遵循 MIT License。同步的文件来自 [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)，请参考其许可证。

## 支持

- 📖 查看详细文档见 `docs/` 文件夹
- 🐛 遇到问题查看 `docs/troubleshooting.md`
- 💬 欢迎反馈和改进建议

## 相关资源

- [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
