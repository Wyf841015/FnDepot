# FNLogPush 飞牛NAS日志哨兵

> 自动监控飞牛NAS系统日志和备份进度，实时推送至多种渠道

[![Platform](https://img.shields.io/badge/platform-FNOS-blue)](https://www.fnnas.com/)
[![Version](https://img.shields.io/badge/version-0.9.30-green)](https://gitee.com/wyf1015/FNLogPush)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

## 功能特性

### 📊 日志监控
- 实时监控飞牛NAS系统日志
- 关键字过滤和告警匹配
- 支持多种日志类型

### 🎯 事件管理
- 独立事件管理面板
- 添加、编辑、删除自定义事件
- 从80+ Font Awesome图标库选择图标
- 支持自定义事件颜色

### 💾 备份监控
- 监控备份任务进度
- 备份完成通知
- 备份历史记录

### 📢 多渠道推送
- 企业微信机器人
- 钉钉机器人
- 飞书机器人
- Bark推送（iOS）
- PushPlus推送
- MeoW推送
- 魔法推送（自定义HTTP POST）
- Webhook自定义

### 🔔 告警聚合
- 智能告警聚合
- 重复告警合并
- 免打扰模式

### 🔒 安全特性
- Session超时自动退出（5分钟无操作）
- 安全的Cookie配置
- 敏感信息加密存储

### 🎨 界面特性
- 8个精选主题（深色/深海蓝/清新绿/暮色橙/科技感霓虹/暗夜紫/暗夜绿/夕阳橙）
- 响应式设计 + 移动端优化
- 实时WebSocket推送

## 安装方式

1. 在飞牛NAS应用中心下载安装
2. 或访问 [Gitee发布页](https://gitee.com/wyf1015/FNLogPush/releases) 下载FPK

## 版本历史

### v0.9.30 (2026-05-09)
- 🐛 **修复推送渠道配置热重载失效** - LogMonitor注册热重载回调，热重载时同步更新push_service和push_coordinator.config
- 🐛 **修复禁用渠道静默失败** - `_execute_push`增加`ch.enabled`检查
- 🐛 **修复update_config竞态条件和configure_from_config线程安全问题**
- 🐛 **修复登录过期返回302重定向而非JSON**
- ✨ **新增5个事件类型** - SSH登录认证失败/登出、Docker初始化异常、系统分区使用率告警/恢复
- ✅ **新增34个测试用例**覆盖核心逻辑

### v0.9.0 (2026-04-19)
- ✨ 统计图表优化与美化、事件分类优化、Bug修复、WebSocket优化

### v0.8.0 (2026-04-06)
- 🔔 事件管理功能 - 独立事件管理面板

### v0.7.0 (2026-04-05)
- 🔒 Session超时机制 - 5分钟无操作自动退出

### v0.6.0 (2026-03-26)
- ✨ 新增钉钉、飞书、Bark、PushPlus推送渠道

### v0.5.0
- 初始版本

## 许可证

MIT License
