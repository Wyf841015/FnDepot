# USB Rsync — Node.js 版 (v0.4.2)

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

### v0.4.2

- **修复**: 导出配置无响应 —— 触发下载的 `<a>` 未插入 DOM，CEF/fnOS WebView 不触发游离元素的 click。现改为插入 DOM → click → 延迟清理，并补充成功/失败提示
- **修复**: 目录浏览在 `package` 用户下不可用（详见下方说明），已恢复 `root` 运行
- **修复**: 设备误识别 —— 原判定只匹配 `/dev/sd*` + `vfat/ntfs/exfat/ext`，把 NAS 自身的 SATA 盘识别成 USB（实测 `/dev/sdc2` 系统盘 ext4 曾被误报为 USB 设备）。改用内核权威属性 `/sys/block/<name>/removable`（`1` = 可移动/USB，`0` = SATA 固定盘），并额外排除挂载在 `/volN` 下的 fnOS 内部卷
- **改进**: 导出配置 / 下载日志改为**先选目录再保存到 NAS**（`/api/save_file`），不再落到浏览器默认下载目录；文件名固定由服务端决定以防目录穿越，目标目录须通过路径白名单校验
- **新增**: 顶部「联系作者」按钮（主题按钮之后），弹窗展示 QQ 群号，支持点击群号或按钮复制
- **改进**: 复制走三级降级（clipboard → execCommand → 自动选中），成功/失败均有文字反馈，适配 fnOS WebView 非安全上下文
- **新增**: 顶部按钮统一规格（34px 高 / 9999px 圆角 / 12px 字号 / 600 字重）并补齐图标，主题按钮带文字（🌙 主题 / ☀️ 浅色）

> **关于运行权限**：本应用曾尝试改为 `package` 用户运行以降低 Web 层风险，实测不可行并已回退。
> 原因是 fnOS 的用户数据目录采用 `000 + ACL` 权限模型（`/vol1/1000` 的 ACL 为 `user::rwx group::--x other::--x`，
> 不含任何应用账号条目），`package` 落入 `other` 只有 `--x` —— 可进入目录但**无法 `readdir` 列目录**。
> 而本应用的核心功能（选择源/目标目录、rsync 增量比对）都依赖列目录，因此必须以 `root` 运行。
> 如需降低风险，建议改为**限制 Web 端暴露面**（绑定 `127.0.0.1` + 远程走 SSH 隧道），而非降权运行。

### v0.4.1 (Node.js 重写)

- **架构**: bash CGI + bash 守护 → 纯 Node 单进程（HTTP + daemon + USB 监控 + 计划调度，零 npm 依赖）
- 保留全部功能: 任务/自动同步/计划/双向/恢复/对比/保留历史/exFAT/通知
- **新增**: 源/目标可用性实时检测（USB 拔出判定）+ **中断自动补同步**（rsync 失败且非用户停止 → 记 pending，USB 重新挂载后 daemon 自动补同步，最多 3 次）
- **修复**: rsync exit 23 的"部分传输"误判（USB 拔出导致源/目标不可用时正确判失败，不再误报完成）

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)