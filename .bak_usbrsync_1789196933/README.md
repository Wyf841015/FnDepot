# USB Rsync CGI

> USB 存储设备自动同步工具，检测到 USB 设备插入后自动同步指定目录，支持单向和双向同步模式。

[![Platform](https://img.shields.io/badge/platform-FnOS-blue)](https://www.fnnas.com/)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

## v0.4.0 更新

- 任务名称编辑、复制及按名称排序
- 插入 USB 自动同步独立开关，支持按 UUID 绑定指定设备
- 同步前试运行，统计新增、修改、删除项目并显示明细
- 每日/每周定时同步
- 任务级排除规则、失败自动重试及重试间隔
- 同步前目标空间检查，支持 rsync checksum 完整校验
- 同步完成/失败后发送 fnOS 系统通知
- 历史详情新增耗时、文件数、传输量、退出码和尝试次数
- 配置导入/导出、日志查看/下载、历史清空
- 存储空间仪表、同步统计 KPI 和移动端完整适配

## v0.2.0 更新

- **修：P0-1 删除 `cmd/main` 空壳** — manifest 删 `service_port`（纯 CGI 不需要常驻进程）
- **修：P0-2 `read_body` 走 `BODY_FILE` 环境变量** — 命令替换 stdin 丢失导致 POST body 永远空的 bug
- **修：P0-3 `/api/tasks` 路由分 GET/POST** — 之前 GET 永远 405
- **修：P0-4 `/api/browse` 路径白名单** — 之前可读任意系统路径（`/etc` 等），现在只允许 `/vol /mnt /media /tmp`
- **修：P0-5 `/api/drives` 改 `/vol*` 路径** — 之前 `for vol in /mnt/vol*` 在 fnOS 上永远空
- **修：P1-1 `dd bs=1` 改 `head -c`** — 1MB body 性能提升 100~500ms
- **修：P1-2 `BASE_DIR` 用 `SCRIPT_FILENAME` 反推** — 不再硬编码 `/var/apps/USBRsyncCgi/target`
- **修：P1-5 拆 `sync.pid`（实际 rsync PID）和 `sync_status.json`（同步状态）** — 新增 `POST /api/sync_stop` 接口
- **修：P1-7 双向同步反向移除 `--delete`** — 之前反向时 `cmd[@]` 包含 --delete，会误删源
- **修：P1-9 同步监控逻辑** — 初始 idle 不再立刻 `clearInterval`；UI 加"⏹ 停止同步"按钮
- **修：P1-10 删除 `authToken` 死代码** — 后端不校验，前端发 token 无意义
- **修：P1-11 `http_error` 改用 `exit 0` 一致** — body 仍输出但 status 头由 http_error 提供
- **修：P1-12 `append_history` 加 `flock` 串行化** — 防并发写丢失
- **修：同步时 status I/O 节流** — 进度变化 ≥1% 才写
- **修：去 5 处 `python3` subprocess** — 用 `grep -oP` 提取 JSON 字段（exclusions/source/target）
- **修：CSS `content:;` 改 `content:''` 伪元素语法** + 移动端 768/480 适配
- **修：path traversal 用 `realpath` 解析后比对** — 不再依赖 `*..*` 字符串匹配
- **修：删除 WeCom webhook 凭据** — install_callback / upgrade_callback 走 A 方案（停用推送）
- **新增：TDD 测试 16/16 通过** — `tests/test_api.js` 覆盖所有关键 API

## v0.1.0 更新

- **初始版本** — USB 自动同步工具，纯 Bash CGI 重写

## 功能特性

- **USB 自动检测** — 实时监控 `/proc/mounts` 和 `lsblk`，自动发现已挂载的 USB 存储设备（vfat/ntfs/exfat/ext4 等）
- **多任务管理** — 支持创建、编辑、删除多个同步任务，灵活配置源目录和目标目录
- **单向/双向同步** — 支持单向同步和双向同步模式
- **删除同步** — 可选 `--delete` 模式，目标目录中与源不一致的文件将被删除
- **目录浏览** — 内置目录浏览器，直接从文件系统选择同步目录
- **实时进度** — 同步过程中实时显示传输进度、当前文件、速度
- **同步历史** — 完整记录每次同步结果
- **排除规则** — 支持自定义排除规则（临时文件、系统文件等）
- **轻量架构** — 纯 Bash CGI 实现，无 Python/Flask 依赖（设备扫描和 rsync 进度解析除外）

## 界面预览

- KPI 卡片展示设备连接数和任务数
- 设备列表展示已挂载 USB 设备（名称、挂载点、文件系统、大小）
- 任务列表支持启用/禁用开关、目录选择、同步模式配置
- 实时同步进度条（百分比、当前文件、传输速度）
- 同步历史记录

## 技术架构

```
USBRsyncCgi/
├── manifest               # FnOS 应用清单
├── cmd/
│   ├── main              # Shell 启动/停止脚本
│   ├── install_init      # 安装钩子
│   ├── install_callback  # 安装回调
│   ├── upgrade_init      # 升级钩子
│   ├── upgrade_callback  # 升级回调
│   ├── uninstall_init    # 卸载钩子
│   ├── uninstall_callback # 卸载回调
│   ├── config_init       # 配置初始化
│   └── config_callback   # 配置回调
├── config/
│   ├── privilege        # 权限配置
│   └── resource         # 资源配置
└── app/
    ├── api.sh            # Bash CGI API（核心业务逻辑）
    └── ui/
        ├── index.cgi      # CGI 入口，路由 /api/* 和静态文件
        ├── config         # FnOS URL 配置
        ├── ICON*.PNG      # 应用图标
        └── www/
            ├── index.html       # 前端入口
            ├── css/style.css    # 样式
            └── js/main.js       # 前端逻辑
```

### 后端 API

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/status` | GET | 心跳检测，返回版本信息、设备数、任务数、同步状态 |
| `/api/devices` | GET | 扫描已挂载 USB 设备 |
| `/api/tasks` | POST | 保存同步任务配置 |
| `/api/sync` | POST | 立即执行同步任务（后台） |
| `/api/sync_status` | GET | 获取当前同步进度 |
| `/api/history` | GET | 获取同步历史记录 |
| `/api/browse` | GET | 浏览目录（path 参数） |
| `/api/drives` | GET | 列出可用存储卷 |

## 构建与打包

```bash
# 打包 fnpack（项目根目录执行，输出 FPK 到当前目录）
fnpack build .

# 输出文件
USBRsyncCgi.fpk
```

## 安装

将 `App.Native.USBRsyncCgi.fpk` 上传至 FnOS 应用中心安装。

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v0.1.0 | 2026-04-30 | 初始版本，纯 Bash CGI 重写 |
| v0.4.0 | 2026-07-25 | 增加设备绑定、计划、预览、重试、校验、通知、统计及配置管理 |

## 维护者

- 作者：[@一零一二](https://gitee.com/wyf1015)
- 主页：https://gitee.com/wyf1015/usbrsynccgi

---

> 如果这个项目对您有帮助，欢迎赞助支持 ❤️
