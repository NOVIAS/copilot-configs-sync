<!-- 
  Quick Start Guide for GitHub Copilot Configs Sync Project
  保存此文件以快速参考项目结构和使用说明
-->

# 项目快速参考 (Quick Reference)

## 📁 项目结构

```
copilot-configs-sync/
├── .github/
│   └── workflows/
│       └── sync-copilot-configs.yml    ← ⭐ 核心 Workflow 文件
├── docs/                                ← 📖 详细文档
│   ├── setup.md                         ← 设置指南
│   ├── configuration.md                 ← 配置说明
│   └── troubleshooting.md               ← 故障排除
├── README.md                            ← 项目说明（必读）
├── SYNC_README.md                       ← 同步说明模板
├── .gitignore                           ← Git 忽略规则
├── package.json                         ← 项目元信息
├── LICENSE                              ← MIT 许可证
└── QUICK_START.md                       ← 本文件
```

## 🚀 5 分钟快速开始

### 1️⃣ 获取 Workflow 文件

复制 `.github/workflows/sync-copilot-configs.yml` 到你的仓库同位置。

### 2️⃣ 推送到 GitHub

```bash
git add .github/workflows/sync-copilot-configs.yml
git commit -m "chore: add copilot configs sync workflow"
git push origin main
```

### 3️⃣ 手动运行以测试

- 进入仓库 **Actions** 标签
- 选择 **Sync Copilot Configs**
- 点击 **Run workflow**

### 4️⃣ 查看结果

- ✅ 成功后会创建 PR
- 📋 合并 PR 应用同步内容
- 📅 每天北京时间 00:00 自动运行

就这么简单！✨

## 📚 完整文档导航

| 文档 | 用途 | 何时查看 |
|------|------|--------|
| [README.md](README.md) | 项目全面介绍 | 初次使用 |
| [docs/setup.md](docs/setup.md) | 详细设置步骤 | 遇到设置问题 |
| [docs/configuration.md](docs/configuration.md) | 高级配置选项 | 需要自定义 |
| [docs/troubleshooting.md](docs/troubleshooting.md) | 问题排查 | Workflow 失败 |

## ⚙️ 默认配置一览表

| 配置项 | 默认值 | 改法 |
|--------|--------|------|
| **运行时间** | 每天北京时间 00:00 | 编辑 cron 表达式 |
| **同步源** | github/awesome-copilot main | 修改 DEFAULT_SOURCE_REF |
| **同步分支** | automation/sync-copilot-configs | 修改 SYNC_BRANCH |
| **自动合并** | 开启（checks 通过后自动合并） | 可删除 auto-merge 步骤 |

## 🎯 常用操作速查

### 立即运行同步

```bash
# GitHub CLI（如已安装）
gh workflow run sync-copilot-configs.yml -R YOUR_USERNAME/YOUR_REPO

# 或通过网页：Actions > Sync Copilot Configs > Run workflow
```

### 检查运行状态

```bash
# 查看最新 10 次运行
gh run list -R YOUR_USERNAME/YOUR_REPO -w sync-copilot-configs.yml --limit 10

# 查看最新运行详情
gh run view --latest -R YOUR_USERNAME/YOUR_REPO
```

### 查看同步的文件

```bash
# 列出同步的指令
ls .github/instructions/

# 列出同步的代理
ls .github/agents/

# 查看 .gitignore
cat .gitignore

# 统计文件数
find .github/{instructions,agents} -type f | wc -l
```

## 🔑 核心概念解释

### 同步分支 (automation/sync-copilot-configs)
- workflow 的工作分支，存储接收同步的改动
- 自动创建 PR 到默认分支
- **用户无需手动操作此分支**

### 自动合并
- PR 创建后自动启用 auto-merge
- 当所有 status checks 通过时自动合并到默认分支
- 合并后同步分支自动删除
- 适合默认分支受保护的仓库

## ⚡ 工作流程示意

```
每天 00:00 (北京时间)
    ↓
[自动检测] awesome-copilot 有无更新
    ↓
[有更新] 克隆最新文件到临时分支
    ↓
[提交] 同步分支推送更改
    ↓
[PR] 自动创建或更新 Pull Request
    ↓
[自动] 启用 auto-merge
    ↓
[checks 通过] 自动合并到主分支
```

## ❓ 快速 Q&A

**Q: 需要手动配置什么吗？**  
A: 不需要！默认配置开箱即用。仅在需要调整时参考 configuration.md。

**Q: 如果有冲突怎么办？**  
A: workflow 使用 rsync --delete，一般不会有冲突。若遇到，参考 troubleshooting.md。

**Q: 可以禁用自动运行吗？**  
A: 可以，移除 cron 触发器，仅保留 workflow_dispatch（手动触发）。


**Q: 怎样防止自定义文件被覆盖？**  
A: 不要在同步目录存放自定义文件。建议使用 `.github/my-instructions/`。

## 🔗 有用的链接

- 📖 [完整 README](README.md)
- ⚙️ [设置指南](docs/setup.md)
- 🔧 [配置文档](docs/configuration.md)
- 🐛 [故障排除](docs/troubleshooting.md)
- 🌟 [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)
- 📚 [GitHub Actions 文档](https://docs.github.com/en/actions)

---

## 📞 需要帮助？

1. 查看相关文档（README.md 或 docs/ 文件夹）
2. 在 [troubleshooting.md](docs/troubleshooting.md) 查找类似问题
3. 提交 Issue 并附加详细日志

---

**记住**: 大多数配置都有合理的默认值。先用默认配置跑起来，再根据需要调整！🎉
