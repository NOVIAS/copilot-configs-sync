# 故障排除 (Troubleshooting)

本文档帮助你快速诊断和解决 Copilot Configs Sync 工作流中遇到的问题。

## 快速诊断指南

### 第 1 步：检查 Action 日志

1. 进入仓库 **Actions** 标签
2. 选择 **Sync Copilot Configs** workflow
3. 找到失败的 run，点击进入详情
4. 展开各 step 查看具体错误信息

### 第 2 步：识别错误类型

根据失败的 step 名称对应本文相关部分：

| 失败的 Step | 对应章节 |
|-----------|--------|
| Checkout repository | [权限问题](#权限问题) |
| Clone source repository | [网络连接](#网络连接问题) |
| Create or update pull request | [PR 相关问题](#pr-相关问题) |
| Create or refresh weekly branch | [周分支问题](#周分支问题) |

---

## 常见问题及解决方案

### ✗ 权限问题

#### 错误信息

```
Error: failed to push to remote.
fatal: Authentication failed
```

或

```
Error: Permission denied (publickey)
```

#### 原因

GITHUB_TOKEN 权限不足或 GitHub Actions 权限配置不当。

#### 解决方案

**1. 检查 Workflow 权限配置**

确保 `.github/workflows/sync-copilot-configs.yml` 包含：

```yaml
permissions:
  contents: write
  pull-requests: write
```

**2. 检查仓库 Actions 权限**

1. 进入仓库 **Settings > Actions > General**
2. 确认 **Actions permissions** 设为 **Allow all actions and reusable workflows**
3. 确认 **Workflow permissions** 为 **Read and write permissions**

**3. 重新运行 Workflow**

修改后重新手动触发一次。

---

### ✗ 网络连接问题

#### 错误信息

```
fatal: unable to access 'https://github.com/github/awesome-copilot.git/': Could not resolve host: github.com
```

#### 原因

1. GitHub runner 无法访问 github.com
2. 网络连接中断
3. DNS 解析失败

#### 解决方案

**1. 检查网络连接**

通常此问题在公网 runner 上不会发生。如果使用自托管 runner，检查网络配置。

**2. 检查 GitHub 服务状态**

访问 [GitHub Status](https://www.githubstatus.com/) 确认服务无故障。

**3. 重试运行**

等待几分钟后手动重新运行 workflow。

---

### ✗ 同步目录不存在或为空

#### 症状

同步完成后，`.github/` 下的同步目录为空或不存在。

#### 原因

1. 源仓库结构改变
2. `rsync` 命令失败
3. 目录权限问题

#### 解决方案

**1. 检查源仓库**

手动访问 https://github.com/github/awesome-copilot 确认文件仍然存在：

```
/chatmodes/
/instructions/
/prompts/
/agents/
```

**2. 查看 rsync 日志**

展开 workflow 中的 "Synchronize directories" step，查看是否有错误信息。

**3. 验证 git sparse-checkout**

检查是否成功获取了子目录：

```bash
# 在日志中查找此行
git sparse-checkout set chatmodes instructions prompts agents
```

---

### ✗ PR 相关问题

#### 错误 1: PR 创建失败

```
Error: GraphQL error: Pull request creation failed
```

**原因**:
- 默认分支受保护且限制了 bot 创建 PR
- 同步分支不存在或被删除

**解决方案**:

1. 检查分支保护规则：**Settings > Branches**
2. 确保允许 GitHub Actions 绕过保护（如启用了）
3. 手动检查 `automation/sync-copilot-configs` 分支是否存在
4. 查看完整日志获取更详细的 GraphQL 错误

#### 错误 2: PR 未创建但无错误

```
Workflow 成功但没有看到 PR
```

**原因**:
- 同步内容没有变化
- PR 已存在且未修改

**解决方案**:

这是正常行为。检查是否：
1. 已有打开状态的 PR 未合并
2. 同步后的内容与当前分支相同

---

### ✗ 周分支问题

#### 错误: 周分支未创建

```
周一运行但没有看到周分支
```

#### 原因

1. 时区判断错误
2. 周分支推送失败
3. 手动运行时不符合条件

#### 诊断步骤

**1. 检查时区判断**

在日志中查找：

```bash
if [[ "$(TZ=Asia/Shanghai date +%u)" == "1" ]]; then
  is_monday=true
```

确认输出为 `is_monday=true`（仅在周一）。

**2. 验证分支推送**

查看是否有推送分支的日志：

```bash
git push --force-with-lease origin "$branch_name"
```

**3. 检查现有周分支**

在 GitHub 界面的 Branches 标签查看是否存在周分支。

#### 解决方案

**手动创建周分支**:

1. 手动运行 workflow
2. 勾选 `create_weekly_branch` = `true`
3. 运行应该会创建周分支

**强制推送周分支**:

```bash
# 本地测试周分支创建逻辑
TZ=Asia/Shanghai date +%u  # 输出 1 表示周一

current_day=$(TZ=Asia/Shanghai date +%d)
current_year_month=$(TZ=Asia/Shanghai date +%Y%m)
echo "${current_year_month}-第$(( ((10#$current_day - 1) / 7) + 1 ))周"
```

---

### ✗ 合并 PR 时冲突

#### 错误信息

```
This branch has conflicts that must be resolved
```

#### 原因

同步分支与默认分支之间有冲突（少见情况）。

#### 解决方案

**1. 手动检查冲突**

```bash
# 本地测试
git fetch origin
git checkout automation/sync-copilot-configs
git merge origin/main
```

**2. 解决冲突文件**

一般只会在 `.github/` 下的配置文件冲突。

**3. 强制合并**

如果 PR 中的文件是自动生成，可以强制覆盖：

```bash
git checkout --theirs .github/
git add .github/
git commit -m "resolve: use synced config"
git push
```

---

### ✗ 内容重复或丢失

#### 症状

同步后发现文件重复、数量变少或内容错误。

#### 原因

1. `rsync --delete` 意外删除了文件
2. 自定义文件误放在同步目录
3. 并发运行导致冲突

#### 解决方案

**1. 不要在同步目录存放自定义文件**

使用单独目录存放自定义配置：

```
.github/
├── chatmodes/           ← 同步目录
├── instructions/        ← 同步目录
├── prompts/             ← 同步目录
├── agents/              ← 同步目录
├── my-instructions/     ← 你的自定义
└── custom-skills/       ← 你的自定义
```

**2. 从 Git 历史恢复**

```bash
# 查看历史
git log --oneline .github/

# 恢复到指定版本
git checkout <commit> -- .github/
```

**3. 禁用并发防护机制**

检查是否启用了 `concurrency` 限制：

```yaml
concurrency:
  group: sync-copilot-configs
  cancel-in-progress: false
```

---

## 高级诊断

### 启用调试模式

编辑 workflow，在 bash steps 添加 `set -x`：

```yaml
run: |
  set -x  # 打印所有执行的命令
  set -euo pipefail
  ...
```

### 本地模拟 Workflow

在本地 Ubuntu/WSL 环境测试 workflow 中的命令：

```bash
# 模拟克隆和同步
git clone --depth=1 --branch main --filter=blob:none --sparse \
  https://github.com/github/awesome-copilot.git test-repo

cd test-repo
git sparse-checkout set chatmodes instructions prompts agents

# 检查新同步的文件
ls -la chatmodes/
ls -la instructions/
```

### 查看完整日志

GitHub Actions 日志可能被截断。尝试：

1. 使用 GitHub CLI 下载完整日志：

```bash
gh run view <run-id> --log
```

2. 在 Actions 页面点击 **More actions > Download logs**

---

## 获取帮助

### 收集诊断信息

报告问题时，请提供：

1. **Workflow 运行 ID 和链接**
2. **完整错误日志**（从 Actions 页面复制）
3. **你的 workflow 配置**（删除敏感信息）
4. **仓库结构**（执行 `find .github -type f | head -20`）
5. **系统环境**（如使用自托管 runner，提供系统信息）

### 常用命令排错

```bash
# 检查 Git 配置
git config -l | grep user

# 查看本地分支
git branch -a

# 查看远程分支
git ls-remote origin

# 测试 GitHub 连接
ssh -T git@github.com

# 查看最近提交
git log --oneline -5

# 检查状态
git status
```

---

## 常见错误代码

| 代码 | 含义 | 解决方案 |
|------|------|--------|
| 1 | 通用错误 | 查看详细日志 |
| 128 | Git 错误 | 检查 Git 配置或网络 |
| 127 | 命令不存在 | 检查 Runner 环境 |
| 2 | 权限/身份验证错误 | 查看权限配置 |
| 22 | HTTP 状态错误 | 检查网络和防火墙 |

---

## 预防措施

### 1. 定期监控
- 每周检查一次 Actions 页面
- 订阅仓库通知

### 2. 备份配置
```bash
git tag backup-config-$(date +%Y%m%d)
git push origin backup-config-$(date +%Y%m%d)
```

### 3. 文档维护
- 保持本文档更新
- 记录自定义配置

### 4. 测试变更
- 修改 workflow 前创建测试分支
- 手动运行以验证

---

## 更多资源

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [awesome-copilot Issues](https://github.com/github/awesome-copilot/issues)
- [GitHub Status Page](https://www.githubstatus.com)
- [Our Project Issues](https://github.com/YOUR_USERNAME/copilot-configs-sync/issues)

---

**最后更新**: 2026-03-30

如果以上方案都无法解决问题，请在项目提交 Issue。
