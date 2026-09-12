# USB Rsync — Node.js 版 (v0.4.1)

> USB 存储设备自动同步工具，检测到 USB 设备插入后自动同步指定目录，支持单向和双向同步模式。
> **Node.js 重写版** — 由 bash CGI + bash 守护架构迁移为纯 Node.js 单进程架构。

## 架构变化 (vs 原 CGI 版)

| 原版 (usbrsync-cgi) | Node.js 版 (usbrsync) |
|---------------------|----------------------|
| bash CGI (`index.cgi` + `api.sh`) | **Node HTTP server** (`ui/server.js`) |
| bash `cmd/main` 守护 (mount_monitor) | **Node 守护** (`ui/daemon.js`, 内嵌于 server) |
| 状态文件共享 (sync.pid/status.json) | 进程内状态 + 磁盘持久化 |
| 每请求起 bash 进程 | 常驻 HTTP server (socket 模式) |
| `ui/server.js` 单进程 = HTTP + USB 监控 + 计划调度 | — |
| rsync 执行 `ui/rsync.js` | — |

**核心优势**：
- 单进程，无 CGI 进程开销，事件驱动并发安全
- USB 监控、计划调度、HTTP 服务内存共享状态，无磁盘竞争
- 原生 `spawn` 管理 rsync 子进程，进度/停止/双向更可靠
- fnOS 标准 socket 模式部署

## 功能 (与原版完全一致)

- **任务 CRUD** — 添加/编辑/复制/删除/排序同步任务
- **自动同步** — USB 插入检测 + UUID 绑定设备 + `auto_sync_on_insert`
- **计划同步** — 每日/每周定时 (cmd 配置)
- **同步模式** — 单向/双向、`--delete`、`--checksum`、重试+间隔、空间检查
- **保留历史** — `--backup --backup-dir=.rsync-history/<时间戳>` (keep_history)
- **反向恢复** — 任务卡「恢复」按钮，备份目标 → 源
- **源/目标对比** — 「对比」按钮 + 同步后自动差异写入历史
- **exFAT/NTFS 兼容** — 自动 `--modify-window=1`
- **预览** — dry-run 统计新增/修改/删除
- **日志** — sync.log 实时追踪 + 5MB 轮转
- **历史** — 100 条上限 + 成功/失败/字节/耗时
- **统计** — KPI 仪表盘
- **fnOS 通知** — 同步完成/失败发桌面铃铛
- **配置导出/导入** — JSON
- **响应式 UI** — 桌面+移动端

## 结构

```
├── manifest            # fnOS 应用清单 (desktop_uidir=ui, nodejs_v24)
├── cmd/main            # 启动脚本 (bridge TRIM→TRM, 起 node server)
├── ui/
│   ├── config          # 网关 socket 配置 (/app/usbrsync)
│   ├── server.js       # 入口: HTTP server + 路由 + 静态 + 启动
│   ├── rsync.js        # rsync 核心: 命令构建/执行/进度/重试/双向/停止
│   ├── daemon.js       # USB 监控守护 + 计划调度
│   └── www/            # 前端 (index.html/css/js)
├── config/             # fnOS 权限/资源
└── tests/              # 测试套件
```

## 开发与测试

```bash
# 本地运行 (端口模式)
TRM_PKGVAR=/tmp/usbrsync PORT=47999 node ui/server.js

# 端到端测试
node --test tests/

# 打包
fnpack build .
```

## 数据目录

- 由 fnOS 注入 `TRIM_PKGVAR` (不改 `@appcenter`, 重装不丢)
- `config/tasks.json` — 同步任务
- `history.json` — 同步历史
- `sync_status.json` — 实时状态
- `logs/sync.log` — rsync 日志

## 版本历史

### v0.4.1 (Node.js 重写)

- **架构**: bash CGI + bash 守护 → 纯 Node 单进程（HTTP + daemon + USB 监控 + 计划调度，零 npm 依赖）
- 保留全部功能: 任务/自动同步/计划/双向/恢复/对比/保留历史/exFAT/通知
- **新增**: 源/目标可用性实时检测（USB 拔出判定）+ **中断自动补同步**（rsync 失败且非用户停止 → 记 pending，USB 重新挂载后 daemon 自动补同步，最多 3 次）
- **修复**: rsync exit 23 的"部分传输"误判（USB 拔出导致源/目标不可用时正确判失败，不再误报完成）

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)