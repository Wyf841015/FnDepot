# FNLogPush 飞牛NAS日志哨兵

> 自动监控飞牛NAS系统日志和备份进度，实时推送至多种渠道

[![Platform](https://img.shields.io/badge/platform-FNOS-blue)](https://www.fnnas.com/)
[![Version](https://img.shields.io/badge/version-1.0.1-green)](https://gitee.com/wyf1015/FNLogPush)
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

### v1.0.1 (2026-06-01)
- ✨ **恢复飞书多维表格统计推送** - 提取公共库 `cmd/lib/feishu_push.sh`
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
