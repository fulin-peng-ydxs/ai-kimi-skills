---
name: operation-docs
description: 响应 operation-docs 调用，使用 operation-workflow 的单位手册模式创建或更新组件、实例及集群的运维手册。
---

# 运维文档入口

本文维护 `operation-docs` 到 `operation-workflow` 单位手册模式的调用方式。运维流程、输出契约及目录约定由 `operation-workflow` 统一维护。

1. 在当前工具已安装技能中查找 `operation-workflow/SKILL.md`；未安装时读取共享的 [运维操作入口](~/.codex/skills/operation-workflow/SKILL.md)。无法读取时报告依赖缺口并停止依赖该技能的操作，不能自行选择输出目录或绕过审批。
2. 以“单位手册模式”处理原需求，保留已确认目标、范围及授权；读取 `operation-workflow` 的单位手册契约。已有证据足够时只编辑文档，需要服务器补充探查时遵循连接和命令授权。
3. 按 `operation-workflow` 完成手册验证、结果审批和 Git 留痕；不为编写手册启停、部署或修改服务器状态。
