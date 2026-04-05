# FNLogPush 飞牛NAS日志哨兵

> 自动监控飞牛NAS系统日志和备份进度，实时推送至多种渠道

[![Platform](https://img.shields.io/badge/platform-FNOS-blue)](https://www.fnnas.com/)
[![Version](https://img.shields.io/badge/version-0.8.0-green)](https://gitee.com/wyf1015/FNLogPush)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

## 功能特性

### 📊 日志监控
- 实时监控飞牛NAS系统日志
- 关键字过滤和告警匹配
- 支持多种日志类型

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
- Webhook自定义

### 🔔 告警聚合
- 智能告警聚合
- 重复告警合并
- 免打扰模式

### 🔒 安全特性
- Session超时自动退出（5分钟无操作）
- 安全的Cookie配置

### 🎨 界面特性
- 5个精选主题（暗夜紫/深色/深海蓝/清新绿/暮色橙）
- 响应式设计 + 移动端优化
- 实时WebSocket推送
- **加载动画**：页面进度条、骨架屏
- **交互动效**：按钮波纹、卡片悬浮、图标脉冲
- **状态指示**：在线状态发光、脉冲动画
- **字体系统**：标题/正文/代码分层

## 安装方式

1. 在飞牛NAS应用中心下载安装
2. 或访问 [Gitee发布页](https://gitee.com/wyf1015/FNLogPush/releases) 下载FPK

## 版本历史

### v0.8.0 (2026-04-05)
- 🎨 **界面全面升级**：字体系统、动画效果、状态指示器
- 📊 添加页面加载进度条 + 骨架屏加载
- 🎬 卡片入场动画 (fadeInUp + stagger)
- 💫 按钮点击波纹 + 悬浮上浮效果
- 📦 卡片悬浮动效 (translateY + shadow)
- ✨ 输入框聚焦/悬浮效果
- 📱 模态框缩放动画
- 🔔 Toast 通知滑动动画
- 🎯 状态指示器脉冲发光
- 🎨 自定义滚动条样式
- ♿ prefers-reduced-motion 无障碍支持

### v0.7.0 (2026-04-05)
- 🔒 **新增Session超时机制**：5分钟无操作自动退出登录

### v0.6.9 (2026-03-29)
- 🆕 新增 MeoW 推送渠道

### v0.6.8 (2026-03-29)
- 🆕 新增存储管理事件、防火墙事件

### v0.6.7 (2026-03-28)
- 🔧 Web界面优化、JS模块化拆分

### v0.6.6 (2026-03-27)
- ✨ WebSocket实时推送

### v0.6.0 (2026-03-26)
- ✨ 新增钉钉、飞书、Bark、PushPlus推送渠道

### v0.5.0
- 初始版本

## 许可证

MIT License
