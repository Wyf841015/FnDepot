# 网络监控 (netwatch)

> 实时网络流量监控：网卡速率、进程级流量归因、连接列表、历史曲线与 CSV 导出，内置 pktstat-BPF 引擎。

![网络监控](ICON_256.PNG)

## v0.6.5 更新

**减少写盘：SQLite 落库从「每次采样」改成「攒批」**

- 🐛 以前 `computeRate()` 内部直接 UPSERT 累计值，而它被「2 秒轮询 + 首次 SSE 连接 + API 兜底」多路径调用，等于每次采样都提交一个事务 —— 实测持续写盘 **23.2 KB/s**，`netwatch.db-wal` 大小不变但 mtime 一直在跳
- ✅ 拆成「算」和「写」：`computeRate()` 只更新内存，落库统一由 `flushCumulativeIfDue()` 决定 —— 距上次 **≥5 秒** 或 累计增长 **≥1 MB** 才写一次（大流量时仍及时落盘）
- ✅ `SIGTERM` / `SIGINT` 退出前强制落一次，崩溃/重启不丢最后一段增量
- ⚡ SQLite 加 `PRAGMA synchronous = NORMAL`（WAL 官方推荐档）+ `temp_store = MEMORY`
- 📉 实测（同一进程改前改后）：**23.2 KB/s → 3.1 KB/s（−87%）**
- 说明：`/proc/<pid>/io` 的 `syscw` 含 stdout/pipe 等非磁盘写，不能当磁盘写次数看；磁盘写次数要看具体文件 mtime 变化（`db-wal` 实测 0.47 次/秒）

## 功能特性

- **实时速率**：总吞吐 / 收 / 发，秒级刷新（SSE 推送，不用轮询）
- **进程级流量归因**：基于 pktstat-BPF（eBPF），按进程统计收发字节，不依赖 nethogs
- **连接列表**：`ss` 解析出的活跃连接，含对端地址、进程、收发字节
- **历史与告警**：SQLite 存 90 天历史点，速率/连接数阈值触发 SSE 告警 + 可选 webhook
- **数据导出**：CSV 导出
- **低资源**：V8 堆限制 96 MB，常驻内存约 130 MB

## 快速上手

1. 安装后打开应用，首页即实时速率与网卡列表
2. 「App 流量」页看进程级归因（首次需要几秒采集）
3. 需要历史曲线时点「历史」，可选时间范围后导出 CSV
4. 告警阈值在「设置」里配（速率 Mbps / 连接数 / webhook）

## 技术架构

```
/proc/net/dev ──┐
                ├── 采样（2s）── 内存累计 ──┬── SSE 推送前端
pktstat-bpf ────┤                          └── SQLite（5s 或 1MB 攒批落库）
ss -tunapie ────┘
```

- 单进程 Node.js（零第三方运行时依赖），fnOS 统一网关 socket 接入
- 累计值跨重启保留（SQLite `cumulative` 表 + `history_points` 历史表）

## 版本历史

### v0.6.5 (2026-10-23)

## v0.6.5 更新

**减少写盘：SQLite 落库从「每次采样」改成「攒批」**

- 🐛 以前 `computeRate()` 内部直接 UPSERT 累计值，而它被「2 秒轮询 + 首次 SSE 连接 + API 兜底」多路径调用，等于每次采样都提交一个事务 —— 实测持续写盘 **23.2 KB/s**，`netwatch.db-wal` 大小不变但 mtime 一直在跳
- ✅ 拆成「算」和「写」：`computeRate()` 只更新内存，落库统一由 `flushCumulativeIfDue()` 决定 —— 距上次 **≥5 秒** 或 累计增长 **≥1 MB** 才写一次（大流量时仍及时落盘）
- ✅ `SIGTERM` / `SIGINT` 退出前强制落一次，崩溃/重启不丢最后一段增量
- ⚡ SQLite 加 `PRAGMA synchronous = NORMAL`（WAL 官方推荐档）+ `temp_store = MEMORY`
- 📉 实测（同一进程改前改后）：**23.2 KB/s → 3.1 KB/s（−87%）**
- 说明：`/proc/<pid>/io` 的 `syscw` 含 stdout/pipe 等非磁盘写，不能当磁盘写次数看；磁盘写次数要看具体文件 mtime 变化（`db-wal` 实测 0.47 次/秒）

### v0.6.4

- 放弃降权，改回 run-as: root（降权后 eBPF 能力永远设不上）

### v0.6.2

- 卸载重装后 App 流量页只剩 (kernel)：启动自检 + 多路径自愈 + 界面告警

### v0.6.0

- App 流量页自动刷新（不必切换页面才有新数据）

### v0.5.0

- pktstat-BPF 引擎替代 nethogs，进程级流量归因

## 维护者

一零一二 · 问题反馈：https://gitee.com/wyf1015/netwatch/issues
