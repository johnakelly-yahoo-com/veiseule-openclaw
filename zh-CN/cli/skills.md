---
summary: "`openclaw skills` 的 CLI 参考（列出/信息/检查）和 skill 资格"
read_when:
  - 你想查看哪些 Skills 可用并准备好运行
  - 你想调试 Skills 缺少的二进制文件/环境变量/配置
title: "skills"
---

# `openclaw skills`

检查 Skills（内置 + 工作区 + 托管覆盖）并查看哪些符合条件，哪些缺少要求。

相关内容：

- Skills 系统：[Skills](/tools/skills)
- Skills 配置：[Skills 配置](/tools/skills-config)
- ClawHub 安装：[ClawHub](/tools/clawhub)

## 命令

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```

