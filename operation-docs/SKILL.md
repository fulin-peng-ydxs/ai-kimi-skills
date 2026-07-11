---
name: operation-docs
description: 运维文档生成与管理
---

# 运维文档生成与管理

## 用途

当用户要求为某个中间件或服务生成/更新运维文档时，以生产环境顶级运维工程师的视角，探查服务器上该服务的实际安装位置、启停方式、配置文件、数据目录、日志位置及常见运维操作，并将总结写入 `~/agent/operation/docs/{服务名称}.md`。

## 使用场景

- 用户说：“帮我整理一下 Nacos 的运维文档”
- 用户说：“给 Redis 写个运维手册”
- 用户说：“总结下这个 docker stack 的维护方式”
- 任何涉及中间件/服务运维知识沉淀的需求

## 执行步骤

1. **确认服务名称**：
   - 从用户输入中提取服务名（如 `nacos`、`redis`、`mysql`、`mongodb`、`kafka`、`zookeeper`、`prometheus`、`grafana`、`nginx`、`docker-stack/xhd` 等）。
   - 如果名称不明确，先询问用户。

2. **探查服务现状**（只读优先，谨慎操作）：
   - 进程：`ps aux | grep -i {服务名}`
   - 端口：`ss -tlnp` / `netstat -tlnp`
   - 安装目录：常见路径 `/opt/{服务名}`、`/usr/local/{服务名}`、`/etc/{服务名}`、`/var/lib/{服务名}`、`/var/log/{服务名}`
   - Docker 相关：`docker ps -a | grep -i {服务名}`、`docker images | grep -i {服务名}`、`docker stack ls`、`docker service ls`、`docker-compose ps`
   - 系统服务：`systemctl list-unit-files | grep -i {服务名}`、`service --status-all | grep -i {服务名}`
   - 启动脚本：`find / -name "startup.sh" -o -name "shutdown.sh" -o -name "*{服务名}*.service" 2>/dev/null`
   - 配置文件：根据服务类型定位核心配置文件

3. **生成运维文档**：
   - 文件路径：`~/agent/operation/docs/{service_name}.md`
   - **服务名称必须使用英文小写**，例如：`nacos`、`redis`、`mongodb`、`mysql`、`kafka`、`zookeeper`、`prometheus`、`grafana`、`nginx`、`docker-stack-xhd`。
   - 文档必须包含以下章节：
      - **服务概述**：服务用途、版本、部署方式
      - **安装位置**：程序目录、配置文件目录、数据目录、日志目录
      - **启停方式**：启动命令、停止命令、重启命令、查看状态命令
      - **端口与进程**：监听端口、进程特征、健康检查方式
      - **常见运维操作**：重启、日志查看、备份（如有）、配置重载、常见问题
      - **依赖与影响**：依赖的其他服务、重启/停止可能影响的上游/下游服务
      - **注意事项**：权限要求、数据安全、生产环境风险提示

4. **写入后验证**：
   - 读取生成的文件，确认格式正确、内容完整。
   - 向用户报告文档路径和核心结论。

5. **更新全局记忆**（如尚未写入）：
   - 确保 `~/.kimi-code/AGENTS.md` 中包含“服务操作优先查阅运维文档”的规则。

6. **联动更新总览文档**：
   - 新增或移除服务子文档后，应调用 `server-operation-index` 技能重新生成 `~/agent/operation/docs/index.md`。
   - 若用户直接要求“生成服务器运维总览文档”或“整理 index.md”，则优先使用 `server-operation-index` 技能，而不是单独生成某个服务文档。

## 安全约束

- 探查阶段以只读命令为主，避免执行启停操作。
- 如需执行非查询命令，必须先评估影响并遵循 `AGENTS.md` 中的权限处理原则（非当前用户目录使用 `sudo`）。
- 不得在生成文档过程中擅自修改服务状态。

## 示例

用户要求：“整理 Nacos 运维文档”。

Agent 应：
1. 检查 `ps aux | grep nacos`、`ss -tlnp | grep 8848`、/opt/nacos、docker 容器等。
2. 生成 `/home/devman/agent/operation/docs/nacos.md`。
3. 报告结果。