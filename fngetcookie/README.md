# 🍪 fngetcookie — Cookie 提取器 (v0.2.1)

> 通过代理方式自动捕获网站 Cookie 的工具，纯 Node.js 零依赖。

## 简介

输入目标网站登录地址，在嵌入式浏览器中完成登录，自动捕获所有 `Set-Cookie`。支持复制 Netscape / JSON / Header 三种格式，适配 curl、wget、yt-dlp 等工具。

**核心特性：**
- 自动捕获：登录即捕获，无需手动查找
- 三格式复制：Netscape（curl/wget/yt-dlp 兼容）、JSON、HTTP Header
- 手动添加：粘贴 cookie 字符串或逐条添加
- 直连模式：抖音等 strict-dynamic CSP 站点可用直连绕过代理
- Cookie 导入：粘贴 Netscape / JSON / Set-Cookie 直接解析
- 站点预设：10 个主流网站一键配置
- 搜索过滤 + 批量操作（多选/全选/批量删除）
- 文件导入导出 + 过期彩色标记
- Cookie 持久化：应用重启后自动恢复
- 会话测试：用已捕获 Cookie 重放请求验证登录态
- 响应式布局：桌面 + 移动端自适应
- 深色/浅色主题
- 安全加固：SSRF 防内网、CORS 同源校验、静态资源白名单

## 安装

1. 在飞牛 NAS 应用中心 → 手动安装 `fngetcookie.fpk`
2. 启用应用，浏览器访问桌面图标

## 维护者

- 作者：[@再见一零一二](https://gitee.com/wyf1015)
- GitHub：[@Wyf841015](https://github.com/Wyf841015)
- Gitee：[@wyf1015](https://gitee.com/wyf1015)