# 配置说明

本文档详细说明如何自定义 Copilot Configs Sync workflow 的各项参数。

## 工作流配置

### 修改同步时间表

编辑 `.github/workflows/sync-copilot-configs.yml` 的 `schedule` 部分：

```yaml
on:
  schedule:
    - cron: '0 16 * * *'  # 当前：每天 16:00 UTC（北京时间 00:00）
```

#### Cron 表达式说明

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
│ │ │ │ │
│ │ │ │ │ 
0 16 * * *
```

#### 常用时间设置

| 含义 | Cron 表达式 | 北京时间 |
|------|------------|--------|
| 每天中午 12 点 | `0 4 * * *` | 12:00 |
| 每天下午 3 点 | `0 7 * * *` | 15:00 |
| 每天晚上 8 点 | `0 12 * * *` | 20:00 |
| 每周一上午 8 点 | `0 0 * * 1` | 周一 08:00 |
| 每月 1 号 0 点 | `0 16 1 * *` | 每月 1 号 00:00 |

### 修改同步源

编辑 `.github/workflows/sync-copilot-configs.yml` 的 `env` 部分：

```yaml
env:
  DEFAULT_SOURCE_REF: main              # 默认同步分支
  SOURCE_REPO: https://github.com/github/awesome-copilot.git  # 源仓库
  SYNC_BRANCH: automation/sync-copilot-configs   # 本地同步分支名
```

#### 使用特定版本

将 `DEFAULT_SOURCE_REF` 改为具体的 tag：

```yaml
env:
  DEFAULT_SOURCE_REF: v1.0.0  # 同步特定版本
```

## 推荐的自定义配置方案

### 方案 1：最小化配置（推荐）

直接使用项目提供的 workflow，最简单：

```yaml
# 保持默认，仅检查权限是否正确
permissions:
  contents: write
  pull-requests: write
```

### 方案 2：定制同步时间

根据你的开发工作时间调整：

```yaml
schedule:
  - cron: '0 4 * * *'  # 改为北京时间中午 12 点
```

### 方案 3：多次循环同步

如果希望一天多次同步（如关键时段）：

```yaml
schedule:
  - cron: '0 4 * * *'   # 北京时间 12:00
  - cron: '0 8 * * *'   # 北京时间 16:00
```

### 方案 4：按需同步（仅手动）

如果不需要自动同步，移除定时触发器：

```yaml
on:
  workflow_dispatch:    # 仅保留手动触发
    inputs: ...
```

## 高级配置

### 配置默认分支检测

如果你不使用 `main` 作为默认分支，需要修改 PR 基准分支：

检查 workflow 中的相关行：

```yaml
- name: Create or update pull request
  ...
  run: |
    ...
    gh pr create \
      --base "${{ github.event.repository.default_branch }}"  # 自动检测默认分支
```

该配置会自动使用仓库设置中的默认分支。

### 修改同步分支名

如需更改同步分支名（默认为 `automation/sync-copilot-configs`）：

1. 编辑 workflow 中的 `SYNC_BRANCH` 变量
2. 更新 PR 相关步骤中的分支引用

```yaml
env:
  SYNC_BRANCH: chore/sync-copilot  # 改为自己的分支名
```

### 禁用自动合并

如果不需要 auto-merge 功能，删除或注释最后一个 step：

```yaml
# - name: Enable auto-merge on pull request
#   if: steps.commit_sync.outputs.changed == 'true'
#   ...
```

## 环境变量配置

所有重要参数都通过环境变量定义，便于维护：

| 变量 | 默认值 | 说明 | 可修改 |
|------|--------|------|--------|
| `DEFAULT_SOURCE_REF` | `main` | 同步源分支 | ✅ |
| `SOURCE_REPO` | `https://github.com/github/awesome-copilot.git` | 源仓库 | ✅ |
| `SYNC_BRANCH` | `automation/sync-copilot-configs` | 同步分支 | ✅ |

## Git 设置

Workflow 自动配置 Git：

```bash
git config user.name "github-actions[bot]"
git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
```

如需改 committer 信息，编辑 workflow 中的这两行。

## 权限配置

### 最小权限原则

```yaml
permissions:
  contents: write        # 必需：推送分支和提交
  pull-requests: write   # 必需：创建和编辑 PR
```

### 更严格的权限配置

如果想要更细粒度的权限控制（不推荐，因为会限制功能）：

```yaml
permissions:
  contents: write
  pull-requests: write
  # 可选：添加其他需要的权限
  # issues: read
  # deployments: write
```

## 条件执行

### 仅在有变化时创建 PR

Workflow 已默认实现此功能：

```yaml
- name: Create or update pull request
  if: steps.commit_sync.outputs.changed == 'true'  # 仅当有变化时执行
```

### 仅工作日运行

若只想在工作日运行（如不需要周末同步）：

```yaml
- name: Resolve source ref and schedule flags
  ...
  run: |
    ...
    day_of_week="$(TZ=Asia/Shanghai date +%u)"
    if [[ "$day_of_week" -ge 1 && "$day_of_week" -le 5 ]]; then
      should_run=true
    else
      should_run=false
    fi
```

### 防并发运行

已默认配置防止多个 run 同时执行：

```yaml
concurrency:
  group: sync-copilot-configs
  cancel-in-progress: false  # 不取消已运行的
```

## 调试配置

### 启用详细日志

在任何 `run` step 添加 shell 选项：

```bash
set -x  # 打印每条命令
```

### 条件 Debug

输出中间变量值用于调试：

```bash
echo "source_ref=$source_ref"
echo "branch_name=$branch_name"
```

## 安全最佳实践

### 1. 限制 Workflow 读写权限

确保仅有必要的权限：

```yaml
permissions:
  contents: write
  pull-requests: write
```

### 2. 使用 `force-with-lease`

Workflow 已使用此安全选项推送分支：

```bash
git push --force-with-lease origin "$SYNC_BRANCH"
```

### 3. 监控 Actions 运行

定期检查 Actions 页面，确保没有异常运行。

## 常见配置问题

### Q: 如何修改 PR 标题？

**A:** 编辑 workflow 中的 `--title` 参数：

```bash
gh pr create \
  ...
  --title "chore: sync latest awesome-copilot files"  # 改为你的标题
```

### Q: 如何跳过自动提交？

**A:** 不推荐，但可以注释掉 "Commit synced changes" step，但这会导致 workflow 无法工作。

### Q: 如何通知团队新的同步？

**A:** 在 SYNC_README.md 中添加通知，或在 workflow 增加通知 step（需额外配置）。

## 配置验证

修改 workflow 后，可通过以下方式验证：

1. **YAML 语法检查**：使用 VS Code 的 YAML 插件
2. **提交并测试**：推送到分支，手动运行一次
3. **查看日志**：检查 Actions 日志确认参数生效

## 恢复默认配置

如果配置出现问题，可以重新从项目获取原始 workflow：

```bash
# 从本项目重新获取
curl -o .github/workflows/sync-copilot-configs.yml \
  https://raw.githubusercontent.com/YOUR_USERNAME/copilot-configs-sync/main/.github/workflows/sync-copilot-configs.yml
```

然后根据需要再次进行自定义。
