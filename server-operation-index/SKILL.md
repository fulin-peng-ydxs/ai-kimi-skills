---
name: server-operation-index
description: 基于服务器实际探测与已有服务文档生成运维总览，记录服务用途、运行单元、端口、依赖、数据及维护入口；不推断未证实的依赖或执行服务变更。
---

# 服务器总运维文档生成

## 用途

当用户需要为当前服务器生成或更新全局运维总览文档时，自动探测服务器上部署的中间件、Docker 服务、数据库、业务应用及其依赖关系，生成 `~/agent/operation/docs/index.md`，并作为各服务子文档的索引入口。

## 使用场景

- 用户说：“生成服务器运维总览文档”
- 用户说：“帮我整理 index.md”
- 用户说：“更新总运维文档”
- 用户说：“给这个服务器建立运维登记册”
- 新服务器首次接入，需要快速沉淀全局运维视图

## 执行步骤

### 1. 收集服务器元信息

先确定目标主机、可访问范围和输出位置，再按实际可用工具选择以下只读命令；进程参数和配置可能含秘密，输出前脱敏，工具缺失不能视为服务不存在：

```bash
# 主机名与操作系统
hostname
cat /etc/os-release
uname -a

# Docker 环境（如存在）
docker version --format '{{.Server.Version}}'
docker info --format '{{.Swarm.ControlAvailable}} {{.Swarm.LocalNodeState}}'
docker stack ls
docker service ls
docker ps -a

# 运行中的系统服务
systemctl list-units --type=service --state=running --no-pager

# 监听端口
ss -tlnp

# 常见中间件进程
ps aux | grep -E 'nacos|redis|nginx|zookeeper|kafka|mysql|mongodb|prometheus|grafana|emqx|tdengine|minio|java' | grep -v grep
```

提取字段：

| 字段 | 来源 |
|------|------|
| `{{hostname}}` | `hostname` |
| `{{os_info}}` | `/etc/os-release` 的 `PRETTY_NAME` + `uname -r` |
| `{{docker_version}}` | `docker version` |
| `{{swarm_role}}` | `docker info` Swarm 状态 |
| `{{probe_time}}` | 当前时间，格式 `YYYY-MM-DD HH:MM` |

### 2. 扫描已有子文档

列出所选输出目录（默认 `~/agent/operation/docs/`）下属于目标主机、除 `index.md` 之外的 `.md` 文件，生成“子文档索引”表格。

| 列 | 说明 |
|----|------|
| 服务名 | 从文件名推断，如 `nacos.md` → Nacos |
| 文档路径 | 相对链接，如 [`nacos.md`](./nacos.md) |
| 说明 | 从该 md 文件的第一段或 `# 服务概述` 提取一句简介 |

若所选输出目录不存在，在落盘时创建该目录。

### 3. 探测运行中的服务

结合系统服务、Docker stack/service、监听端口和进程，识别以下常见服务类型：

- **宿主机服务**：Nacos、Redis、Nginx、Zookeeper、Kafka、MySQL、DM8、TDengine、Java 进程等
- **Docker 服务**：MongoDB、EMQX、MinIO、Prometheus、Grafana、Node Exporter、cAdvisor、业务微服务等
- **Docker Stack**：`docker stack ls` 列出的每个 stack 及其 service 数量

对识别出的每个服务，整理字段：

| 字段 | 说明 |
|------|------|
| 服务名 | 标准名称及其业务用途、调用方 |
| 版本 | 探测到的版本号，未知则写“待确认” |
| 部署方式 | 实际运行单元及管理方式，区分常驻服务、一次性任务、宿主机调度和构建工具 |
| 程序目录 | 安装路径 |
| 启动方式 | 启动命令或 systemctl 服务名 |
| 停止方式 | 停止命令 |
| 监听端口 | 从 `ss -tlnp` 提取 |
| 配置文件 | 主配置文件路径 |
| 数据目录 | 数据/持久化路径 |
| 运维文档 | 链接到实际存在的同主机子文档；尚未生成的只写“待补充”，不生成失效链接 |

### 4. 生成端口矩阵

根据 `ss -tlnp` 和已识别服务，生成端口矩阵表格：

| 端口 | 协议 | 服务 | 类型 | 说明 |
|------|------|------|------|------|
| 22 | TCP | SSH | 系统 | 远程管理 |
| ... | ... | ... | ... | ... |

### 5. 梳理依赖关系

根据配置引用、部署编排或实际调用证据绘制依赖图，解释箭头是调用、数据流还是启动依赖；仅共处一台主机或符合业务常识不能证明依赖。以下只是表达示意，不得直接复制为事实：

```text
业务应用
    ├─▶ Nacos
    ├─▶ Redis
    ├─▶ MongoDB
    ├─▶ MySQL/DM8
    ├─▶ Kafka/EMQX
    └─▶ ...
```

仅列出当前服务器上实际存在的服务，不要虚构。

### 6. 填充模板并写入

使用本技能目录下的 `template.md` 作为蓝本，将第 1-5 步收集的数据替换占位符。模板说明不进入交付正文，无关章节可省略；保留证据边界和实际未决问题。

优先使用用户指定路径或同主机已有总览；否则使用 `~/agent/operation/docs/index.md`。多环境必须区分主机归属，不能覆盖另一环境的总览。

### 7. 验证与报告

- 读取生成的 `index.md`，检查读者能否识别业务入口、运行单元、调用与数据方向、故障影响及对应维护入口；模板示例与占位符必须全部替换或移除。
- 分开说明可复用方案与本机实际状态，标注探测时间、证据范围和差异；命令未执行须标为未验证，进程存在不等于业务健康，无连接快照不等于长期无人调用。
- 向用户报告文档路径、探测到的服务数量、子文档数量、待确认项。
- 若生成了新文件或做了重大更新，按 `AGENTS.md` 要求追加 `~/agent/operation/operation-log.md`。

## 模板变量说明

`template.md` 中使用的占位符：

| 占位符 | 含义 | 必填 |
|--------|------|------|
| `{{hostname}}` | 服务器主机名 | 是 |
| `{{os_info}}` | 操作系统信息 | 是 |
| `{{docker_version}}` | Docker 版本 | 有则填 |
| `{{swarm_role}}` | Swarm 角色 | 有则填 |
| `{{probe_time}}` | 探测时间 | 是 |
| `{{runtime_diagram}}` | 实际运行单元、业务入口和数据方向图 | 是 |
| `{{special_mechanisms}}` | 实际特殊部署机制，无则说明或移除章节 | 有则填 |
| `{{operation_entry_table}}` | 带前置条件、结果和验证状态的维护入口 | 是 |
| `{{sub_docs_table}}` | 子文档索引 Markdown 表格 | 是 |
| `{{service_list}}` | 中间件与基础服务清单 | 是 |
| `{{stack_table}}` | Docker Stack 清单表格 | 有则填 |
| `{{port_matrix}}` | 端口矩阵表格 | 是 |
| `{{dependency_diagram}}` | 服务依赖关系 ASCII 图 | 是 |
| `{{data_backup_table}}` | 数据目录与备份重点表格 | 是 |
| `{{pending_items}}` | 已知问题与待确认项 | 是 |
| `{{change_log}}` | 变更与维护记录 | 是 |

对于表格类占位符，直接生成完整 Markdown 表格文本；对于列表类，生成条目或章节。

## 输出规范

- 文档语言：与用户当前对话语言一致。
- 子文档命名：`{服务英文名}.md`，全部小写。
- 路径：遵循上述输出优先级，所有子文档和模板链接按实际位置解析。
- 状态未知项：明确写“待确认”或“未探测到”，不臆造。
- 敏感信息：若探测输出包含密码、Token、密钥，写入前必须脱敏。

## 安全约束

- 探测阶段仅使用只读命令，禁止执行启停、删除、写入类操作。
- 对 `/opt`、`/etc`、`/var` 等系统目录的查看需评估影响，必要时使用 `sudo` 但保持只读。
- 不得修改任何正在运行服务的配置或状态。

## 维护联动

- 新增/移除子文档后，应重新生成本文档，更新“子文档索引”和“中间件与基础服务清单”。
- 每次重大变更后，更新“变更与维护记录”。
- 状态快照按 `AGENTS.md` 要求归档到 `~/agent/operation/data/`。