# 自动同步说明

此仓库中的 Copilot 配置文件通过 GitHub Actions 自动从 [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot) 同步。

## 同步内容

- **.github/chatmodes/**: GitHub Copilot 聊天模式文件
- **.github/instructions/**: GitHub Copilot 指令文件
- **.github/prompts/**: GitHub Copilot 提示文件
- **.github/agents/**: GitHub Copilot 代理文件

## 同步时间

- 每天北京时间 00:00 自动执行同步（UTC 16:00）
- 每周一北京时间 00:00 的同步结果可自动创建周归档分支

## 手动触发

如需立即同步，可在 GitHub Actions 页面手动运行 **Sync Copilot Configs** workflow。

支持的手动输入参数：

| 参数 | 示例值 | 说明 |
|------|--------|------|
| `source_ref` | `main` 或 `v1.0.0` | 同步源的分支/tag |
| `create_weekly_branch` | `true` | 强制创建当前周的归档分支 |

## 同步流程

1. 克隆 awesome-copilot 源仓库（只下载需要的目录）
2. 同步 chatmodes、instructions、prompts、agents 到 `.github/` 下
3. 更新并提交到同步分支 `automation/sync-copilot-configs`
4. 自动创建或更新 PR 到默认分支
5. 每周一时，从同步分支创建周归档分支（如启用）

## 注意事项

### ⚠️ 请勿直接修改同步目录中的文件

以下目录的内容会在同步时被覆盖：

```
.github/chatmodes/
.github/instructions/
.github/prompts/
.github/agents/
```

### 如何定制 Copilot 配置

如需自定义配置，请在其他目录维护自己的文件：

- **仓库级自定义指令**: `.github/copilot-instructions.md`
- **路径特定指令**: `.github/instructions/my-custom.instructions.md`

示例结构：

```
.github/
├── chatmodes/              ← 自动同步（勿改）
├── instructions/           ← 自动同步（勿改）
├── prompts/                ← 自动同步（勿改）
├── agents/                 ← 自动同步（勿改）
├── copilot-instructions.md ← 你的自定义内容
├── my-instructions/        ← 你的自定义目录
│   └── custom.instructions.md
└── my-skills/              ← 你的自定义技能
    └── my-skill/
        └── SKILL.md
```

## 来源项目

所有同步的文件来源于：

- **源仓库**: [github/awesome-copilot](https://github.com/github/awesome-copilot)
- **同步内容**: 官方提供的教程、指令、代理和提示
- **更新频率**: 通过此项目每日自动同步

## 周归档分支说明

每周一自动创建周归档分支，命名规则为 `YYYYMM-第N周`：

- `202603-第1周` - 2026年3月第1个周一的同步内容快照
- `202603-第2周` - 2026年3月第2个周一的同步内容快照
- ...以此类推

周分支包含该周最新同步的完整内容，可用于：

- 📌 保存周度快照
- 🔍 追踪内容变化历史
- 🔄 快速回滚到某个周的配置

## 工作流状态检查

### 查看同步日志

1. 进入 GitHub 仓库主页
2. 点击 **Actions** 标签
3. 选择 **Sync Copilot Configs** workflow
4. 点击最近的运行记录查看详细日志

### 常见状态

| 状态 | 含义 |
|------|------|
| ✅ **Success** | 同步完成，有变化时会创建 PR |
| ⏭️ **Skipped** | 已有 PR 未合并，跳过本次同步 |
| ❌ **Failed** | 同步失败，检查日志排查问题 |

## 第一次使用后的文件结构

完成首次同步后，仓库结构会如下所示：

```
.github/
├── workflows/
│   └── sync-copilot-configs.yml
├── chatmodes/
│   ├── backend.chatmode.md
│   ├── frontend.chatmode.md
│   └── ...
├── instructions/
│   ├── typescript.instructions.md
│   ├── python.instructions.md
│   └── ...
├── prompts/
│   ├── test-gen.prompt.md
│   ├── doc-gen.prompt.md
│   └── ...
└── agents/
    ├── code-reviewer.agent.md
    ├── architect.agent.md
    └── ...

SYNC_README.md  ← 每次同步自动更新时间戳
```

## 其他帮助

- 📖 完整配置和使用说明见 [README.md](README.md)
- 📋 详细设置指南见 `docs/setup.md`
- 🔧 配置调整见 `docs/configuration.md`
- 🐛 故障排除见 `docs/troubleshooting.md`

---

**上次更新**: 当 workflow 运行时自动更新
