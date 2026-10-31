---
name: operation-docs
description: 兼容旧 operation-docs 调用名称；收到该旧名称请求时转交 operation-workflow 的单位手册模式，不再独立维护运维流程或模板。
---

# 运维文档兼容入口

本文只保留旧调用名到 `operation-workflow` 的路由，不维护第二套文档能力或默认路径。新任务使用 `operation-workflow`。

1. 在当前工具已安装技能中查找 `operation-workflow/SKILL.md`；未安装时读取共享的 [运维操作入口](~/.codex/skills/operation-workflow/SKILL.md)。无法读取时报告依赖缺口，不能回退到过期全局目录或绕过审批。
2. 以“单位手册模式”处理原需求，保留已确认目标、范围及授权；读取新技能的单位手册契约。已存在证据足够时只编辑文档，需要服务器补充探查时遵循连接和命令授权。
3. 目录、手册最低内容、验证、结果审批和 Git 留痕全部由新技能维护；不为编写手册启停、部署或修改服务器状态。
