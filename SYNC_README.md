# 自动同步说明

此仓库中的 Copilot 配置文件通过 GitHub Actions 自动从 [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot) 同步。

## 同步内容

- **.github/instructions/**: GitHub Copilot 指令文件
- **.github/agents/**: GitHub Copilot 代理文件

## 同步时间

- 每天北京时间 00:00 自动执行同步
- 每周一北京时间 00:00 的同步结果可自动创建周归档分支

## 手动触发

如需立即同步，可在 GitHub Actions 页面手动运行 **Sync Copilot Configs** workflow。

## 注意事项

请勿直接修改上述同步目录中的文件，否则会在下一次同步时被覆盖。如需定制，请在其他目录维护自己的文件。

## 来源项目

- 来源仓库: https://github.com/github/awesome-copilot
- 来源版本: main

最后同步时间: 2026-03-30 09:41:10 UTC
