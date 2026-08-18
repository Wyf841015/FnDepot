# FNLogPush 飞牛NAS日志哨兵

> 自动监控飞牛NAS系统日志和备份进度，实时推送至多种渠道

[![Platform](https://img.shields.io/badge/platform-FNOS-blue)](https://www.fnnas.com/)
[![Version](https://img.shields.io/badge/version-1.3.0-green)](https://gitee.com/wyf1015/FNLogPush)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-110%20passed-brightgreen)](tests/)
[![Python](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org/)

## 功能特性

### 📊 日志监控
- 实时监控飞牛NAS系统日志
- 关键字过滤和告警匹配
- 支持多种日志类型（系统、SSH、存储、备份、Docker 等）

### 🎯 事件管理
- 独立事件管理面板
- 添加、编辑、删除自定义事件
- 从 80+ Font Awesome 图标库选择图标
- 支持自定义事件颜色
- 支持查询数据库中的最新事件记录

### 💾 备份监控
- 监控备份任务进度
- 备份完成通知
- 备份历史记录

### 📦 下载中心监控
- 监控 DownloadCenter 任务状态变化
- 支持完成、卡死、失败、恢复、暂停、开始、新增 7 种事件
- 自动检测下载卡死并告警

### 🖼️ 相册监控
- 监控相册新建、分享、设备注册
- 分享链接实时生成

### 🎬 影视监控
- 监控影视库更新（新增/删除/收藏/播放）

### 🐳 Docker 容器监控
- 通过 `docker` CLI 监控容器状态（兼容无 Socket 权限环境）
- 自动修复 Socket 权限
- 支持启停/创建/销毁/暂停/恢复/健康状态变化事件
- CLI 方案：自动检测 + sudo 兜底

### 📢 多渠道推送
- 企业微信机器人
- 钉钉机器人
- 飞书机器人
- Bark 推送（iOS）
- PushPlus 推送
- MeoW 推送
- 魔法推送（自定义 HTTP POST）
- Webhook 自定义

### 🔔 告警聚合
- 智能告警聚合
- 重复告警合并
- 免打扰模式（DND 期间缓存，结束后统一汇总）

### 🔒 安全特性
- Session 超时自动退出（5 分钟无操作）
- 安全的 Cookie 配置
- 敏感信息加密存储（Fernet）
- CORS 环境变量化（不再 `*` 通配符）
- Docker 容器 ID 验证防命令注入
- `_strip_masked` 占位符回填（凭据编辑不丢原值）

### 🎨 界面特性
- 6 个精选主题（深色 / 深海蓝 / 清新绿 / 暮色橙 / 科技感霓虹 / 暗夜紫）
- 响应式设计 + 移动端优化
- 实时 WebSocket 推送健康状态（fnOS 网关兼容）
- 统计图表（推送趋势、渠道对比、事件类型、小时活动、监控类型、聚合对比）
- 加载动画：页面进度条、骨架屏
- 交互动效：按钮波纹、卡片悬浮、图标脉冲
- 移动端底部菜单 + 同步 active 状态
- 字体系统：标题/正文/代码分层

## 安装方式

1. **在飞牛 NAS 应用中心搜索 "日志哨兵" 下载安装**（推荐）
2. **手动安装**：下载 [fnlogpush.fpk](https://gitee.com/wyf1015/FNLogPush/releases) → 应用中心 → 手动安装

## 开发者

### 环境要求
- Python 3.9+
- Node.js 16+（构建前端）
- Linux x86_64

### 构建步骤
```bash
# 1. 安装 Python 依赖
cd cmd/logmonitor && python3 -m venv venv
source venv/bin/activate && pip install -r requirements.txt

# 2. 构建前端（esbuild + CSS concat）
cd /vol3/1000/hermes/fnlogpush
node build.js   # 生成 bundle.js + bundle.css

# 3. 打包 fpk
fnpack build .  # 输出 fnlogpush.fpk 到项目根目录
```

### 测试
```bash
# 单元测试（110 个，含 docker/download/photo/media/push/config/pydantic/hot-reload/dnd 监控服务）
cd /vol3/1000/hermes/fnlogpush && python3 -m pytest -q tests/

# 合计：110 个测试，全部通过
```

### 项目结构
```
fnlogpush/
├── cmd/
│   ├── install_callback         # 安装时初始化（生成加密密钥、复制默认配置）
│   ├── upgrade_callback         # 升级回调
│   ├── uninstall_callback
│   ├── lib/
│   │   └── feishu_push.sh       # 内部公共库（install/upgrade 共享）
│   └── logmonitor/              # 主源码
│       ├── src/
│       │   ├── main.py
│       │   ├── app.py
│       │   ├── routes/          # API 路由
│       │   ├── services/        # 业务服务
│       │   │   ├── docker_monitor_service.py
│       │   │   ├── download_monitor_service.py
│       │   │   ├── photo_monitor_service.py
│       │   │   ├── media_monitor_service.py
│       │   │   ├── backup_monitor_service.py
│       │   │   ├── push_service.py
│       │   │   └── ...
│       │   ├── monitor/         # DB pollers
│       │   ├── utils/
│       │   ├── config/
│       │   ├── templates/
│       │   │   └── index.html   # 前端入口
│       │   └── static/
│       │       ├── js/          # 前端源码 + bundle.js
│       │       └── css/         # 前端样式 + bundle.css
│       ├── config/              # 默认配置
│       └── tests/               # pytest 单元测试
├── tests/                       # 根目录测试（92 个）
├── build.js                     # 前端构建脚本（esbuild + CSS concat）
├── package.json
├── manifest                     # fnpack 应用清单
└── fnlogpush.fpk                # 打包产物
```

## 版本历史

### v1.3.0 (2026-08-18)
- 新增：系统通知监控（notify_monitor）— 轮询 PostgreSQL trim.notify 表检测新系统通知并推送到已配置渠道
- 新增：Web 设置界面「系统通知」配置面板（启用开关 / 轮询间隔 / 最低级别 / 来源过滤）
- 新增：/api/notify/status + /api/notify/config 路由
- 优化：保存配置后自动启停通知监控线程（无需重启应用）
- 优化：KPI 仪表盘新增通知监控状态显示

### v1.2.2 (2026-08-17)
- ✅ **测试覆盖 88%** — 新增 17 个测试文件，覆盖 52/59 个源码模块（路由层、推送渠道、数据模型、加密工具、核心服务、轮询器等）
- 🐛 **修复 DND 汇总统计为 0** — summary_only 模式下 `clear_cache` 清空 `cached_records` 后 `format_statistics_summary` 返回 total=0，改为预计算 summary_text/record_count 再清空
- 🔧 **修复时区偏移** — 后端 logtime 是 CST 本地 epoch（比标准 UTC 少 8h），`is_timestamp_in_dnd()` 缺 +28800 导致 DND 时段日志全被推送
- 🔧 **2 个 P1 稳定性修复** — 前端 resize 防泄漏、ECharts 面板生命周期 dispose

### v1.2.0 (2026-06-11)
- 🐛 **DND 缓存消息双重推送终结修复** - 模块级 `_DND_FLUSH_LOCK` 替代实例级 `self._flush_lock`，5 个 monitor 各自创建 DNDHandler 实例的竞争根本解决
- 🔐 **P0 安全加固（13 项修 9）**:
  - inline onclick 12 处挂 `window`（静默失败终结）
  - XSS textContent 替代 innerHTML（download.js + main.js）
  - `apiFetch` 错误处理: 解析 body 拿 code，不泄露内部路径
  - `login_required` 从 no-op 改为校验 `X-Trim-UID`
- ♻️ **P0 稳定性修复**:
  - Socket reconnect 回调废弃（死代码 200+ 行清理）
  - Docker socket 拒绝 chmod 666
  - `sleep(N)` → `Event.wait(N)` 消除死休眠
- ⚡ **P1 优化（5 项）**:
  - 删除 5 处重复 `@login_required` 装饰器（-16 行）
  - 6 处 `setInterval` 加 `clearInterval` 防 timer 堆积
  - 生产环境 `console.log` 静默，`localStorage.DEBUG=true` 可恢复
  - `resize` 监听器先 remove 再 add 防泄漏
  - ECharts 实例面板切离时 dispose（释放内存）
- ✅ **测试全覆盖** - 零回归，所有 P0+P1 修复经 pytest + happy-dom 端到端验证
- 🐛 **修复 DND 退出轮内新日志合并** - 主循环顺序 `process_logs → check_dnd_cache` 导致 DND 退出瞬间同一轮既推汇总又推本轮新日志。`check_dnd_cache` 加 `pending_new_logs` 参数 + 返 bool，主循环调换顺序
- 🐛 **修复 DND 缓存消息乐观清空失败** - DND 层保留缓存与 push_service 重试队列竞争导致同条消息推 2 遍。锁内先清空缓存再推送，失败只走 push_service 重试
- 📦 **重打干净 fpk** - 905KB

### v1.1.7 (2026-06-07)
- 🐛 **修复勿扰模式汇总消息反复推送** - 5 个 monitor service 各自实现 `_check_dnd_flush` 共享同一份 `dnd_service.cached_messages` 且无锁 → DND 结束瞬间 5 个 monitor 同时 flush 同一份汇总推到企业 IM 多次。统一改走 `monitor_core/dnd.py DNDHandler.check_dnd_cache` 入口 + `threading.Lock` 锁内原子清空，删 5 份重复实现（-200+ 行死代码）
- 🧪 **新增 2 个并发回归测试** - `test_concurrent_check_dnd_cache_only_pushes_once`（5 线程并发只推 1 次，bug 复发即 FAIL）+ `test_concurrent_failure_preserves_cache`（推送失败缓存不丢）
- 🔄 **重写 4 个 service 自身的 DND 测试** - 从 `assert hasattr(svc, '_check_dnd_flush')` 改为 `assert not hasattr(...)`（反向断言：5 monitor 不应再有自己的 flush，统一入口在 `monitor_core/dnd.py`）
- 🧹 **重打干净 fpk** - 905KB（含 `clean-runtime-artifacts.sh` 清理）

### v1.1.6 (2026-06-06)
- 🐛 **修复 KPI "最后推送" 时间晚 8 小时** - `utils/time_utils.py:timestamp_to_shanghai` 之前 `+timedelta(hours=8)` 后又 `astimezone(Asia/Shanghai)` 重复偏移 8h（实测 09:38:04 → 17:37:53）。删 +8h 直接 astimezone
- 🐛 **修复 1.5s setTimeout 无参调用 syncKpiFromStatus bug** - 改 `setTimeout(() => loadStatus(), 1500)` 走 patchLoadStatus 链路（a5086e0 漏修的另一条调用链）
- 🐛 **修复 last_push_time server 重启后归零** - push_service 内存值重启清零导致 KPI 永远 `--`，新增 `HistoryService.get_last_successful_push_time()` 从 push_history 表回退（success=1 最新一条）
- 🧪 **新增 tests/verify-kpi-loadstatus-e2e.js** - happy-dom 端到端 7/7 通过（关键坑：mock fetch 必须用 `new window.Response(JSON.stringify(obj), {...})`，plain object 的 .json() 在 happy-dom 抛错走 catch 干扰测试）

### v1.1.5 (2026-06-05)
- 🎨 **主题系统 P0 5 项全修复** - 主题白名单单一真理源 / 统一 CSS 选择器前缀 / 修复 initTheme race condition / 拆分 ReduceMotionManager 独立模块 / setTheme 同主题跳过 + 循环写防护
- ✨ **主题系统 P1 4 项全完成** - 主题切换 transition CSS (0.3s 渐变) / 缓存 themeClasses 数组 / debounce + requestIdleCallback 后端保存 + beforeunload flush
- 🐛 **真正修复"最后推送为空"bug** - bundle.js 末尾 `return data` + `syncKpiFromStatus` 接收 data 参数，端到端 4/4 验证
- 🧹 **重打干净 fpk** - 加 `scripts/clean-runtime-artifacts.sh` 清理 .pyc/.db/.log/.pytest_cache（fpk 体积 -563KB / -38%）
- 🛠️ **加 `tests/notify_tool/fn_notify.py`** - 飞牛OS 系统通知查询 SQL 模板（参考）+ CLI 发送/清理工具

### v1.1.0 (2026-06-05)
- ⚡ **alert_aggregator 防 OOM** - 单 group 日志累积 cap 在 500 条，长期运行不再内存累积
- ⚡ **photo_monitor 状态写盘优化** - dirty + 60s 后台 flush 替代每次轮询都写 config.json
- 🔧 **loguru console 级别调整为 WARNING** - `info.log` 不再被 INFO 噪声灌满（之前 27MB+ 增长，现在只写 WARN/ERROR/CRITICAL）
- ✨ **新增事件 `APP_INSTALL_FAILED_INSTALL_INIT_EXCEPTION`** - 用于 app 安装初始化失败场景
- 🐛 **修复 Invalid Date 显示** + 若干内部缺陷（UnboundLocalError、bundle.js build 链路）

### v1.0.2 (2026-06-05)
- 🔧 **P0-1 update_config 走 pydantic 严格校验** - `AppConfig.model_validate()` 替代 dict update，新增额外字段 `extra='allow'` 保留兼容
- 🔧 **P0-2 热重载同步所有 service** - 抽 `_apply_config_to_services` 让 `update_config` 和 `_on_config_hot_reload` 共享（DND + alert_agg + 5 monitor 不再漏更新）
- 🔧 **P0-3 备份恢复路径穿越** - 严格文件名白名单 `[a-zA-Z0-9_]+\.json` + `Path.is_relative_to` 校验
- 🔧 **P0-5 根治 KPI 卡片** - 改 `build.js` 手动 concat 共享 IIFE，不再被 esbuild 重命名（`CONSTANTS2` ReferenceError 修复）
- 🏗️ **P1-1a 渠道配置字典化** - 8 渠道从 124 行 if/elif 改为 `_CHANNEL_SPECS` dict 循环处理
- 🏗️ **P1-1b push_channels/ 目录拆分** - `push_service.py` 1538 → 883 行，8 个渠道类拆到子目录
- ♻️ **P1-2 抽 `push_and_record` helper** - photo + download monitor 共享推送逻辑
- 🔒 **P1-3 10 个写/读敏感端点加 `@login_required`** - 配置/备份/恢复/Docker 操作统一认证入口
- ⚡ **P1-4 chart_data N+1 → Counter** - 24×500 条记录从 24 次遍历优化为 1 次
- 🧹 **P2-7.2 裸 `except:` → `except Exception:`** - 避免吞 `KeyboardInterrupt`
- 🧹 **P2-7.3 CONFIG_DIR 注入 app.config** - 6 个端点去 hardcoded fallback
- 🧹 **P2-7.5 删 `validate_webhook_url` 死代码** - 0 调用方，pydantic 已覆盖
- ✅ **测试 92 → 112** - 新增 pydantic 严格校验回归 + hot-reload 同步 + chart_data 桶化 + 备份路径穿越 + 端到端 KPI 验证

### v1.0.1 (2026-06-01)
- ✨ **恢复飞书多维表格统计推送** - 提取公共库 `cmd/lib/feishu_push.sh`，install/upgrade 共享
- 🐛 **修复移动端底部菜单 active 状态不同步** - `syncNavActive` 同步 `.mobile-nav-btn`
- 🐛 **修复 WebSocket 连接失败** - URL 改用 `GATEWAY_BASE` 直接构造，兼容 fnOS 网关
- 🐛 **修复统计图表不显示** - `switchNavPanel` 切换到 stats 面板时主动调用 `initStatsPanel`
- 🐛 **修复测试推送确认框无反应** - HTML id `confirmModalConfirm` → `confirmModalBtn`
- 🐛 **修复备份数据库连接测试"缺少参数"** - HTML id 改为驼峰命名匹配 JS
- 🐛 **修复 Webhook 渠道测试端点错误** - 改用 `/api/test-webhook`
- 🐛 **修复 GATEWAY_BASE 拼接导致 API 失败** - `apiFetch` 正确拼接

### v1.0.0 (2026-05-30)
- 🎨 **暗夜紫主题全面重写** - 对照暮色橙结构，浅色紫色系
- 🗑️ **删除 legacy 树**（`cmd/logmonitor/logmonitor/`）
- 🔢 **版本号统一为 1.0.0**

### v0.9.40 (2026-05-30)
- 🔄 同步最新代码版本

### v0.9.36 (2026-05-11)
- 🎨 主题系统全面优化 - 后端添加白名单校验，修复深色主题检测逻辑
- ✨ 主题自动保存 - `setTheme()` 自动同步到后端
- ⚡ 性能优化 - 批量 `classList.remove()` 替代 forEach
- 🔧 代码复用 - `ThemeManager.isDark()` 统一 11 处深色检测

### v0.9.35 (2026-05-10)
- 🔒 API 响应格式统一 - `/api/health` 添加 `success: true` 字段
- ✅ XSS 防护确认 - 用户数据已使用 `escapeHtml()` 转义
- ✅ 路由分析完成 - `/api/agg/stats` 与 `/api/alert-aggregation/stats` 返回不同数据
- ✅ 刷新逻辑分析 - stats.js 无 setInterval，仅手动刷新
- ✅ 无障碍优化 - aria 属性已完善

### v0.9.34 (2026-05-10)
- 🔒 CORS 配置修复 - 从环境变量 `CORS_ORIGINS` 读取
- 🔒 Docker subprocess 防护 - 容器 ID 格式验证防命令注入
- 🐛 ConfigManager 修复 - 删除重复的 `validate_config` 方法
- 🐛 DND 缓存修复 - `get_cached_messages` 返回副本
- ✨ MagicPush 测试 - 新增 10 个单元测试

### v0.9.33 (2026-05-10)
- 🐛 修复勿扰阶段消息汇总功能失效
- 🐛 修复推送渠道响应判断
- 🐛 修复 Webhook enabled 属性缺失
- 🐛 修复 Bark/PushPlus JSON 解析
- 🐛 修复 Webhook POST 字段名（content → msg）
- ✨ **新增 Docker 容器监控** - 自动检测并修复 Docker Socket 权限
- ✨ Docker 事件支持 DND

### v0.9.30 (2026-05-09)
- 🐛 修复推送渠道配置热重载失效
- 🐛 修复禁用渠道静默失败
- 🐛 修复 `update_config` 竞态条件
- 🐛 修复 `configure_from_config` 线程安全
- 🐛 修复登录过期处理（302 vs 401 JSON）
- ✨ 新增 5 个事件类型（SSH 登录/登出、Docker 初始化异常、系统分区使用率告警/恢复）
- ✅ 新增 34 个测试用例

### v0.9.9 (2026-04-27)
- 🔄 同步至最新代码版本

### v0.9.5 (2026-04-23)
- ✨ 丰富推送信息内容（新建相册/分享/设备注册/批量创建）
- 🐛 分享相册优化
- ⚡ 备份监控优化

### v0.9.0 (2026-04-19)
- ✨ 统计图表优化与美化
- 🔧 事件分类改为下拉选择框
- 🐛 Bug 修复

### v0.8.4 (2026-04-08)
- 🐛 修复 MeoW 推送状态判断

### v0.8.3 (2026-04-08)
- 🐛 修复推送渠道重启后无法解密（预生成加密密钥）

### v0.8.2 (2026-04-08)
- 📝 消息分段推送（解决长消息失败）

### v0.8.0 (2026-04-06)
- 🔔 事件管理功能

### v0.7.6 (2026-04-05)
- 🎨 界面重构：玻璃拟态侧边栏

### v0.7.0 (2026-04-05)
- 🔒 Session 超时机制（5 分钟）

### v0.6.0 (2026-03-26)
- ✨ 新增钉钉、飞书、Bark、PushPlus 推送渠道

### v0.5.0
- 初始版本

## 许可证

MIT License
