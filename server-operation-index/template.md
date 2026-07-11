# 服务器运维总览文档

## 文档说明

本文档是 `{{hostname}}` 服务器的**总运维登记册**，汇总了该服务器上部署的所有中间件、Docker 服务、数据库、业务应用及其依赖关系，并作为各服务子文档的索引入口。

> 探测时间：{{probe_time}}  
> 服务器主机名：`{{hostname}}`  
> 操作系统：{{os_info}}  
> Docker 版本：{{docker_version}}  
> Swarm 角色：{{swarm_role}}

---

## 子文档索引

{{sub_docs_table}}

> 其他中间件的子文档待后续按需补充。

---

## 全局架构概览

```text
┌─────────────────────────────────────────────────────────────┐
│                      {{hostname}} 服务器                       │
├─────────────────────────────────────────────────────────────┤
│  请在生成时根据实际服务绘制 ASCII 架构图                        │
│  示例：                                                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │  Nacos  │  │  Redis  │  │  Nginx  │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
├─────────────────────────────────────────────────────────────┤
│                    Docker 容器 / Swarm                        │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  Stack: xxx (N services running)                       │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 中间件与基础服务清单

{{service_list}}

---

## Docker Stack 清单

{{stack_table}}

> 业务 stack 的详细运维文档应单独生成子文档并登记到上方索引。

---

## 特殊部署机制

> 在此记录非标准部署、定时任务、镜像分发、自动部署脚本等特殊机制。
> 若不存在，可保留本节并写“当前未发现特殊部署机制”，或删除本节。

### （示例）镜像自动分发服务

| 项目 | 内容 |
|------|------|
| 机制名称 | 待填写 |
| 状态 | 运行中 / 已禁用 / 待确认 |
| 触发条件 | 待填写 |
| 执行逻辑 | 待填写 |
| 相关文件 | 待填写 |

---

## 端口矩阵

{{port_matrix}}

---

## 服务依赖关系

```text
{{dependency_diagram}}
```

---

## 运维入口速查

### 查看所有运行中服务

```bash
# 系统服务
systemctl list-units --type=service --state=running

# Docker Stack
docker stack ls
docker service ls

# Docker 容器
docker ps -a

# 监听端口
ss -tlnp

# 当前 Java 进程
ps aux | grep java | grep -v grep
```

### 常用启停命令

> 根据实际探测到的服务，补充常用启停命令。示例：

```bash
# Nacos
sudo /opt/nacos/bin/startup.sh -m standalone
sudo /opt/nacos/bin/shutdown.sh

# Redis
sudo systemctl start|stop|restart redis-server

# Nginx
sudo /opt/nginx/sbin/nginx
sudo /opt/nginx/sbin/nginx -s stop
sudo /opt/nginx/sbin/nginx -s reload

# Docker Stack
# cd /opt/app && sudo docker stack deploy -c docker-stack.yml <stack_name>
```

---

## 数据目录与备份重点

{{data_backup_table}}

---

## 已知问题与待确认项

{{pending_items}}

---

## 变更与维护记录

{{change_log}}

---

## 文档维护规范

- 新增中间件/服务时，应先在本文档中登记，再补充独立子文档到 `~/agent/operation/docs/`。
- 子文档命名规范：`{服务英文名}.md`，全部小写，如 `redis.md`、`nginx.md`、`mongodb.md`。
- 每次重大变更后，更新本文档的“变更与维护记录”章节。
- 状态快照归档到 `~/agent/operation/data/`。