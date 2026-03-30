# 设置指南

本指南将帮助你快速部署和配置 Copilot Configs Sync 项目。

## 前置要求

- GitHub 账户和一个你拥有的仓库
- 基本的 Git 知识
- 仓库已启用 GitHub Actions（默认启用）

## 第 1 步：获取 Workflow 文件

### 选项 A：Fork 本项目

```bash
# 克隆本项目
git clone https://github.com/YOUR_USERNAME/copilot-configs-sync.git
cd copilot-configs-sync

# 推送到你的仓库（如果使用自己的仓库）
git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### 选项 B：仅复制 Workflow 文件

如果你已有仓库，只需复制 `.github/workflows/sync-copilot-configs.yml` 文件：

```bash
# 在你的仓库根目录
mkdir -p .github/workflows
cp /path/to/sync-copilot-configs/sync-copilot-configs.yml .github/workflows/

git add .github/workflows/sync-copilot-configs.yml
git commit -m "chore: add copilot configs sync workflow"
git push
```

## 第 2 步：验证基本设置

### 查看仓库设置

1. 进入你的 GitHub 仓库
2. 点击 **Settings**
3. 在左侧菜单选择 **Actions > General**
4. 确认 **Actions permissions** 为 **Allow all actions and reusable workflows**

### 检查分支保护规则

如果你的默认分支启用了分支保护，需要允许 GitHub Actions 创建 PR：

1. 点击 **Settings > Branches**
2. 选择默认分支的保护规则
3. 在 **Bypass rules** 中添加 **GitHub Actions** 应用

## 第 3 步：首次运行

### 自动运行

workflow 默认配置为每天北京时间 00:00 运行。如果你想立即测试：

### 手动触发运行

1. 进入仓库的 **Actions** 标签
2. 左侧选择 **Sync Copilot Configs** workflow
3. 点击 **Run workflow** 按钮
4. 配置可选参数（保持默认即可）
5. 点击 **Run workflow** 确认

### 查看运行结果

1. 等待几秒，workflow run 会出现在列表中
2. 点击 run 进入详情页
3. 展开各个 step 查看执行过程
4. 成功后会看到：
   - ✅ 绿色成功标记
   - 🔀 创建或更新的 PR（如有变化）
   - 📊 同步统计信息

## 第 4 步：合并 PR（第一次）

首次运行后会自动创建一个 PR，包含同步的所有文件：

1. 进入仓库的 **Pull Requests** 标签
2. 找到标题为 "chore: sync awesome-copilot configs" 的 PR
3. 查看变化内容
4. 点击 **Merge pull request** 合并

## 第 5 步：验证文件结构

合并 PR 后，你的 `.github/` 目录应包含：

```
.github/
├── workflows/
│   └── sync-copilot-configs.yml       ← 工作流文件
├── chatmodes/                         ← 同步内容
├── instructions/                      ← 同步内容
├── prompts/                           ← 同步内容
└── agents/                            ← 同步内容
```

运行以查看同步的内容：

```bash
# 查看同步目录列表
ls -la .github/chatmodes/
ls -la .github/instructions/
ls -la .github/prompts/
ls -la .github/agents/

# 查看总文件数
find .github/{chatmodes,instructions,prompts,agents} -type f | wc -l
```

## 第 6 步：配置自定义指令（可选）

现在你可以在仓库中创建自己的 Copilot 配置，这些不会被同步覆盖：

### 创建仓库级指令

创建 `.github/copilot-instructions.md`：

```markdown
---
description: '你的项目级 Copilot 指令'
---

# 你的项目指令

## 代码风格

- 使用 TypeScript
- 遵循 ESLint 配置
- 注释优先

## 技术栈

- 前端: React + TypeScript
- 后端: Node.js + Express
```

### 为特定文件创建指令

创建 `.github/instructions/typescript.instructions.md`：

```markdown
---
description: 'TypeScript 项目指令'
applyTo: '**/*.ts,**/*.tsx'
---

# TypeScript 编码指南

当编辑或创建 .ts/.tsx 文件时应用此指令...
```

### 创建你的技能

创建 `.github/skills/my-feature/SKILL.md`：

```markdown
---
name: 'My Feature Skill'
description: '你的自定义技能描述'
---

# My Feature Skill

此技能用于...
```

## 运行第二次同步

等待第二天的自动运行，或手动触发以验证设置：

### 预期行为

1. ✅ 成功同步最新的 awesome-copilot 文件
2. ✅ 如果有新文件，自动创建更新 PR
3. ✅ 如果没有变化，workflow 完成但不创建 PR
4. ✅ 周一运行时，自动创建周分支（如已启用）

## 常见问题排排

### 权限不足

**错误**: `Permission denied` 或 `You do not have permission to...`

**解决方案**:
1. 检查仓库 Settings > Actions 权限设置
2. 确认 GITHUB_TOKEN 回复权限包括 `contents:write` 和 `pull-requests:write`

### Network 连接错误

**错误**: `Could not resolve host: github.com`

**解决方案**:
1. 检查 GitHub Actions Runner 能否访问 github.com
2. 如果在内网，可能需要配置代理
3. 查看 workflow 日志获取详细错误信息

### 时间条件未触发

**现象**: workflow 按时间表应该运行，但没有运行

**解决方案**:
1. 确认仓库没有禁用 Actions
2. 检查分支是否推送到了 main（workflow 监视此分支）
3. GitHub Actions 时间表偶尔会有延迟（±5分钟）

### 周分支未创建

**现象**: 周一时没有看到周分支

**解决方案**:
1. 检查是否是通过自动触发运行的（不是手动运行）
2. 查看日志中的 `is_monday` 输出，确认时区判断正确
3. 可以手动运行并勾选 `create_weekly_branch` 参数强制创建

## 下一步

- 📖 查看 [configuration.md](configuration.md) 了解高级配置
- 🔧 查看 [troubleshooting.md](troubleshooting.md) 排查问题
- 💡 参考 [README.md](../README.md) 了解完整功能
