# FNLogPush 飞牛NAS日志哨兵

> 自动监控飞牛NAS系统日志和备份进度，实时推送至多种渠道

[![Platform](https://img.shields.io/badge/platform-FNOS-blue)](https://www.fnnas.com/)
[![Version](https://img.shields.io/badge/version-1.1.8-green)](https://gitee.com/wyf1015/FNLogPush)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

## 功能特性

### 📊 日志监控
- 实时监控飞牛NAS系统日志
- 关键字过滤和告警匹配
- 支持多种日志类型（系统、SSH、存储、备份、Docker 等）

### 🐳 Docker 容器监控
- 通过 `docker` CLI 监控容器状态（兼容无 Socket 权限环境）
- 自动修复 Socket 权限
- 支持启停/创建/销毁/暂停/恢复/健康状态变化事件

### 💾 备份监控
- 监控备份任务进度
- 备份完成通知 + 历史记录

### 📦 下载中心 / 🖼️ 相册 / 🎬 影视监控
- DownloadCenter 任务状态变化（完成/卡死/失败/恢复/暂停/开始/新增）
- 相册新建/分享/设备注册
- 影视库更新（新增/删除/收藏/播放）

### 📢 多渠道推送
- 企业微信、钉钉、飞书机器人
- Bark（iOS）、PushPlus、MeoW
- 魔法推送（自定义 HTTP POST）、Webhook 自定义

### 🔔 告警聚合
- 智能告警聚合 + 重复告警合并
- 免打扰模式（DND 期间缓存，结束后统一汇总）

### 🔒 安全特性
- Session 超时自动退出（5 分钟无操作）
- 敏感信息加密存储（Fernet）
- CORS 环境变量化
- Docker 容器 ID 验证防命令注入
- `_strip_masked` 占位符回填（凭据编辑不丢原值）

### 🎨 界面特性
- 6 个精选主题（深色 / 深海蓝 / 清新绿 / 暮色橙 / 科技感霓虹 / 暗夜紫）
- 响应式设计 + 移动端优化
- 实时 WebSocket 推送健康状态
- 统计图表（推送趋势 / 渠道对比 / 事件类型 / 小时活动 / 监控类型 / 聚合对比）
- 移动端底部菜单

## 安装方式

1. **在飞牛 NAS 应用中心搜索 "日志哨兵" 下载安装**（推荐）
2. **手动安装**：下载 `fnlogpush.fpk` → 应用中心 → 手动安装

## 版本历史

### v1.1.8 (2026-06-08)
- 🐛 修复 DND 退出轮内新日志合并 + 缓存消息乐观清空，彻底解决勿扰模式双推问题

### v1.1.7 (2026-06-07)
- 🐛 修复勿扰模式汇总消息反复推送 - 5 monitor 共享 `dnd_service.cached_messages` 但各自 flush 无锁，DND 结束瞬间 5 monitor 同时 flush 同一份汇总推到企业 IM 多次。统一改走 `monitor_core/dnd.py DNDHandler.check_dnd_cache` + `threading.Lock` 锁内原子清空
- 🧹 删 5 份重复 `_check_dnd_flush` 实现（-200+ 行死代码）
- 🧪 新增 2 个并发回归测试（5 线程并发只推 1 次 + 失败时缓存保留）
- 📦 重打干净 fpk 905KB

### v1.1.5 (2026-06-05)
- 🎨 主题系统 P0 5 项全修复 - 白名单单一真理源 / CSS 选择器前缀统一 / race condition 修复 / ReduceMotionManager 拆分 / 同主题跳过
- ✨ 主题系统 P1 4 项全完成 - transition 0.3s / themeClasses 缓存 / debounce + requestIdleCallback / beforeunload flush
- 🐛 真正修复"最后推送为空"bug - bundle.js 数据传递 + syncKpiFromStatus 端到端验证
- 🧹 重打干净 fpk - 加清理脚本，fpk 体积 -38%

### v1.1.0 (2026-06-05)
- ⚡ **alert_aggregator 防 OOM** - 单 group 日志累积 cap 在 500 条，长期运行不再内存累积
- ⚡ **photo_monitor 状态写盘优化** - dirty + 60s 后台 flush 替代每次轮询都写 config.json
- 🔧 **loguru console 级别调整为 WARNING** - `info.log` 不再被 INFO 噪声灌满（之前 27MB+ 增长，现在只写 WARN/ERROR/CRITICAL）
- ✨ **新增事件 `APP_INSTALL_FAILED_INSTALL_INIT_EXCEPTION`** - 用于 app 安装初始化失败场景
- 🐛 修复 Invalid Date 显示 + 若干内部缺陷

### v1.0.2 (2026-06-05)
- 🔧 update_config 走 pydantic 严格校验
- 🔧 热重载同步所有 service（DND + alert_agg + 5 monitor 不再漏更新）
- 🔧 备份恢复路径穿越 - 严格文件名白名单
- 🔧 根治 KPI 卡片 - bundle.js build 链路修复
- 🏗️ 渠道配置字典化（8 渠道从 124 行 if/elif 改为 dict 循环）
- 🏗️ push_channels/ 目录拆分 - push_service.py 1538 → 883 行
- ♻️ 抽 push_and_record helper
- 🔒 10 个写/读敏感端点加 @login_required
- ⚡ chart_data N+1 → Counter 优化
- 🧹 裸 except → except Exception、删死代码、CONFIG_DIR 注入 app.config
- ✅ 测试 92 → 112

### v1.0.1 (2026-06-01)
- 🐛 修复移动端底部菜单 active 状态不同步
- 🐛 修复 WebSocket 连接失败（URL 用 `GATEWAY_BASE` 构造，兼容 fnOS 网关）
- 🐛 修复统计图表不显示（`switchNavPanel` 主动调用 `initStatsPanel`）
- 🐛 修复测试推送确认框无反应
- 🐛 修复备份数据库连接测试"缺少参数"
- 🐛 修复 Webhook 渠道测试端点错误

### v1.0.0 (2026-05-30)
- 🎨 暗夜紫主题全面重写（浅色紫色系）
- 🗑️ 删除 legacy 树（`cmd/logmonitor/logmonitor/`）
- 🔢 版本号统一为 1.0.0

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
